---
title: '[翻译]Go MySQL 驱动的三个BUG'
tags: 
  - MySQL
  - Go
---

# 翻译前言

在工作场合遇到了`invalid connection`问题，稍微阅读了一下Go MySQL driver的源码后，无意间读到这篇文章。觉得这种细致入微和追根究底的工程精神一直是自己所追求但还是欠缺的，因此决定翻译一下这篇文章，仔细阅读且希望未来自己能成为和作者一样的工程师。

# 正文

尽管Github.com依旧是运行于Rails的一个单体应用（monolith），但在过去的几年中，我们逐渐提取并用Go重写了那些在Rails难以运行地更快且稳定的核心功能。去年，我们在生产环境上部署了一个名为`authzd`的新的验证服务，而该服务很好地响应了我们在GitHub Satellite 2019时宣布的“fine-grained authorizations”功能。

这即是个重要的里程碑也是个有趣的调整，因为`authzd`是我们的第一个可以在生产环境上读取MySQL数据库数据的Go语言服务。在过去，我们也曾在内部环境里部署过Go服务，但是他们大多数只是用于内部管理，比如`manticore`是一个用于搜索集群管理（Search cluster manager）的服务，而`gitbackups`是一个负责定时备份Git repository的异步批处理服务。但这一次，`authzd`将被我们的Rails单体应用在正常运行期间所调用，这意味着我们对性能和可靠性的要求更加严格了。

然而在我们将`authzd`部署到我们的Kubernetes集群后，一个问题出现了。在建立一个新的TCP连接时，延迟会特别高。很明显，有什么东西使得Go MySQL驱动的连接轮询出了问题。程序员总是喜欢告诉自己网络是可靠的（这是一个可怕的谎言），但是，大多数时候网络的确是可靠的。所以当我们看到有些事不太对的时候，比如延迟变高或者网络断连，我们就不得不去找一找是不是我们所依赖的库出了问题（译者注：而非网络的问题）。

而下面就是我们从MySQL的角度去发现问题解决问题的过程 —— 为了确保`authzd`可以支持我们生产环境上的流量并且符合我们的SLO指标。

# 问题 （The crash）
在所有已知的Go MySQL的bug里，`Invalid connection (unexpected EOF)`是最经常遇到且捉摸不定的。在两年前我们第一次部署代码库备份服务`gitbackups`时，我们就遇到了这个问题，分析原因并且提交了BUG反馈。在那时，我们尚不明确该如何修复MySQL驱动里的这个问题，所以我们选择在应用程序的代码里进行处理。去年，当我们第一次在生产环境下部署`authzd`时，这个问题又一次出现并导致了非常高的错误率：每分钟都有上百个`Invalid connection`的错误报告。这使我不得不再一次面对这个问题。这一次，我们终于找到了问题的根源并且修复了它 —— 没有在应用层代码上进行任何改动。让我们深入了解这个问题以及解决方法。

## 问题的修复 （Fixing the issue）
Go SQL驱动为Go的标准库之一`database/sql`包提供了一个基础的组成部分`SQL 连接（SQL connection）`。由于所有主流的SQL服务器（包括MySQL和Postgres）的默认连接行为都是有状态且不可以被同时使用，这个驱动程序提供的每一条连接也是有状态且无法被同时使用。因此，`database/sql`这个包试图提供一个可以管理所有连接的访问（access）与连接时常（lifetime）的功能。这个包里面的`DB`结构（struct）是所有SQL数据库的入口，它是一个连接池（connection pool）的抽象，里面包含了所有该驱动提供的连接。因此，调用诸如`(*DB).Query`或者`(*DB.Exec)`会设计很多复杂的逻辑：首先，这个方法会对连接池进行轮询，试图在里面找到一条可用（active）但是空闲（idle）的连接。当所有连接都处于忙碌状态（busy）时，它会命令底层驱动创建一个新的连接，并且执行数据库指令或者查询，之后将该连接返还连接池。如果此时连接池已经满了（译者注：到达最大连接数）则丢弃该连接。

`unexpected EOF`的问题在`gitbackups`和`authzd`应用代码试图从连接池获得一条连接执行查询时都出现过。通过浏览MySQL服务器的日志我们可以轻松地找到原因：由于SQL服务器与客户端的连接时长超过了最大的空闲超时设置（idle timeout）， SQL服务器会主动关闭在Go的`sql.(*DB)`连接池中的连接。请注意，为了防止在同一时刻有过多的连接，我们特地地设置了一个相对长地空闲超时时长（30秒），但这依旧无法阻止`invalid connection`的错误出现，并且这个问题令我们的服务不断地崩溃。

对于TCP连接来说，中断是很正常的。而`database/sql`包正是为了解决这些问题而设计的：通过特定的驱动程序实现，它理应可以处理在任一时刻，几百条的连接中的任一一条连接的中断。一条连接可以因为多种原因变得不正常（unhealthy）：它是否关闭，是否接收到错误的数据包，又或者是SQL连接可能导致的任意错误中的一种。而当驱动程序检测到这样一种不正常时，任何一个使用该连接的方法都会在被调用时返还`driver.ErrBadConn`。

在`database/sql`包里，`ErrBadConn`是一个非常有用（magic）的错误，它表示该条连接已经不再有效（valid）。如果这条连接依旧在连接池里，它会被移除。如果这条连接被调出执行查询，该条连接会被立即弃用，而新的一条连接会被从连接池中调出或者创建。在知道了这样一种行为后，我们就应该知道调用`(*DB).Query`永远不会遇到`invalid connection`错误，因为即使这个库试图使用一条无效的连接（invalid connection）进行查询，库会返还的错误应该是`driver.ErrBadConn`且该查询会在用户不知情地情况下在另外一条新的连接上执行（译者注：即就算是选到了无效连接，返还地也不是invalid connection错误而是bad connection错误）。

因为，到底是什么使得`authzd`遇到了`invalid connection`错误呢？

TCP协议与位于TCP协议之上的MySQL wire协议有一些细微的不匹配。一条TCP连接是全双工（full-duplex）的，这代表了数据可以从该连接的任意一个方向独立地发送，而位于TCP协议之上地MySQL协议则在进入命令阶段后就完全由客户端管理。MySQL服务器只会给发来数据包地客户端发回数据（进行响应）。这就导致服务器端是有可能由于空闲超时设置或者是`pt-kill`之类的进程将一条有效连接（active）关闭，而此时客户端依旧处于命令阶段。让我们看一下图示：

{% asset_img network_diagram.png 图1:表示在Go MySQL驱动程序和MySQL服务器之间发送的数据包的网络流程图 %}

从这个BUG的复现里，我们可以看到在建立了TCP握手后，MySQL级别的请求和响应随之建立了。一段时间后，由于在超时时间内没有新的请求，MySQL服务器执行了关闭socket的操作，并且服务器内核（kernel）发送了`[FIN, ACK]`用以告知客户端服务器已经完成了数据传送，客户端内核则回应`[ACK]`来告知服务器收到了，但此时客户端的连接没有被关闭（译者注：只有服务器端的连接被关闭了），这就是所谓的TCP的全双工：客户端作为写数据的一方，它的连接是独立的，并且只有在客户端调用`close`才能关闭该连接。

对于大多数建立于TCP协议之上的网络协议，这并不是一个问题。当客户端从服务器读取数据时，只要它接受到了`[SYN, ACK]`信号，它就知道服务器不会继续使用该连接写入更多数据，因此会在下一个读操作时返回`EOF`错误。然而正如之前提到过的，一旦MySQL连接处于命令模式下，MySQL协议就完全是客户端管理的。服务器只会响应来自于客户端的请求，客户端只会在发送请求后才会从服务器读取数据。

上面的网络示意图清晰地展示了实际流程。在经过一段放置后（sleeping period），我们会使用客户端的某一条有效（open）连接进行写入操作，而由于服务器端的连接已经关闭（客户端并不清楚这个事实），它会回复`[RST]`数据包。紧接着，我们的客户端会试图从MySQL服务器读取响应并且得到了`EOF`错误，这时它才知道该连接已经无效（invalid）。

这就是为什么我们的连接崩溃了但是我们的应用并没有崩溃。那么为什么当我们发现一条MySQL连接是无效的时候不返回`driver.ErrBadConn`错误呢？为什么不让`database/sql`试图建立全新的连接重试查询呢？这是因为这样的重试查询并不安全。

网络示意图里所展示的流程在生产环境里发生的很频繁。在MySQL的集群中，每一分钟都发生上千次，特别是在非高峰期时，由于服务大部分时间处于空闲状态，空闲超时很容易被触发。当然，这并非是唯一一个关闭连接的原因。

那么当我们使用一个正常（healthy）连接进行`UPDATE`操作时，网络中断会发生什么呢？Go MySQL驱动程序同样会在执行写命令时收到`EOF`错误。如果这个时候返回的是`driver.ErrBadConn`错误，灾难就会发生：`database/sql`会用一个新的连接执行`UPDATE`，导致数据出错（corruption）！这是非常麻烦的。在通常情况下，当MySQL关闭连接时，重试查询是安全的，我们可以知道之前发送的查询命令并不会被MySQL服务器接收到，更不用担心会被执行。但是我们必须假设一个最坏的情况，就是我们的查询指令确确实实被送到了服务器，也因此，返回`driver.ErrBadConn`是不安全的。

所以，我们该如何修复这个问题？在`gitbackups`中我们使用了一个最简单的解决方法，即调用客户端里的`(*DB).SetConnMaxLifetime`方法对MySQL连接的最大连接时长进行一个设置：把这个数值设置的比MySQL集群的空闲超时小就行了。然而这不是最理想的解决方案。`SetConnMaxLifetime`设置了客户端上的任意一条连接的最长持续时间（maximum duration），而非最长待机时间（maximum idle duration），这就意味着`database/sql`会不停地关闭一些连接，而这些连接有可能长在被使用或者压根没有触发服务器端的最长空闲超时时间。`database/sql`包并没有提供最长待机时间（maximum idle duration）接口（API），因为并不是所有的SQL服务器都有空闲超时的实现，抽象的灾难……也因此，这个解法实际上是可以接受的，只是它导致了不必要的连接中断，以及无法检测到SQL服务器是否在主动关闭减少连接。

显而易见，最佳解决方案是在当我们在执行写入操作时，将每一条从连接池里面得到的连接都进行一次检测，查看运行情况。很不巧的是这只有当Go 1.10引入了`SessionResetter`之后才有可能实现，因为在此之前，我们并没有办法得知什么时候一条连接会被返回连接池，这也是为什么我们等了将近两年才可以对这个程序进行一个合适的修复。

检查一条连接是否正常有两种方法：在MySQL层面上进行检测以及在TCP层面上进行检测。MySQL的运行状况检查包含了发送`PING`数据包并且等待响应。由于`PING`本身不在MySQL上执行任何写操作，所以返回`driver.ErrBadConn`是永远安全的。然而这样有一个缺点，因为所有刚刚从连接池中提取的新连接的第一个操作都会因此而增加延迟。这是个麻烦的问题，因为Go的应用程序中，连接池的提取返回操作是很频繁的。因此，我们选择了更简单的修复方式，即在TCP的层面上检查一个连接是否被MySQL服务器关闭。

这个检查花费很小，我们只需要在尝试任何一个写操作之前就执行一个非阻塞读操作。如果服务器关闭了连接，我们将会收到`EOF`的错误，反而如果连接依旧有效，我们也会马上收到`EWOULDBLOCK`错误来告知我们要读取的数据尚不存在。好消息是，如今Go程序里面的socket（套接字）都已经被设置为了非阻塞模式。不要被Goroutine的优雅抽象所迷惑，它的原理是Go调度器会使用异步事件循环（async event loop）让Goroutine在没有数据需要读取的时候进行睡眠（sleep）并且在有数据时被唤醒（weak）。坏消息是因为调度器会让没有任务的routine进入睡眠模式，我们无法通过调用诸如`net.Conn.Read`等方法来执行非阻塞读操作。到了`Go 1.9`版本，执行这种非阻塞读取操作的接口才被引入：`(*TCPConn).SyscallConn`让我们可以使用底层文件描述符（file descriptor）去调用系统的读操作（syscall.Read)（而不需要经过Go routine的读管道）。

借助这个新的API，我们终于可以以不到5微妙的速度开销进行连接的检查，使之没有“陈旧连接”。这都是因为我们可以使用非阻塞读取 —— 当然，非阻塞读取本身就应该很快，因为它们不会阻塞。

当我们在生产环境里实行了这个修复后，所有的`invalid connection`错误都消失了，我们的服务可用性也终于到达了“9”开头。

{% asset_img client_side_needles.png 图2:部署后authzd的客户端数据 %}

## 小提示 （Production tips）

- 如果你的MySQL服务器使用了空闲超时，或者它自己会主动断开连接，你不需要在生产环境下使用`(*DB).SetConnMaxLifetime`。这并不是必须的，因为驱动可以正常检测并且重试无效（stale）的连接。设置连接的最大生命周期只会导致所有有效连接的不必要的关闭和开启。

- 使用服务高峰期时段的数据来设置`sql.(*DB)`连接池的`(*DB).SetMaxIdleConns)`和`(*DB).SetMaxOpenConn`以及保证MySQL服务器会在空闲期间（off-hours)对空闲连接进行积极的修建（pruning）是管理MySQL高吞吐访问量的一个很好的模式。这些被关闭的连接会被MySQL驱动程序检测并且在必要时重新创建。

# 超时 （The timeout）

当我们把`authzd`这类的服务加入到我们的单体应用的依赖项里的时候，该服务的响应时长也相应地加到了我们服务的所有请求的总响应时长中，或者说，被加入了所有需要进行验证的请求中。也因此，确保`authzd`的请求不会超过我们分配给它的最长处理时间是非常关键的，否则将会导致整个应用的性能下降。

从Rails的角度，这代表我们需要非常小心地处理RPC请求的超时问题。而在Go的服务器端，这意味着一件事，`Context`。`Context`包是在`Go 1.7`中被引入进标准库里的，在此之前它已经作为独立的库被使用多年。它的立意很简单，通过将`context.Context`传入所有与请求生命周期相关的函数里，使之可以在该被取消时可以取消。每一个发过来的HTTP请求都会有一个`Context`对象，如果客户端提前断开了连接，则该`Context`对象也会被取消，甚至可以不取消该对象而是延长它的截止日期（deadline）。这让我们使用一个API更好地管理请求超时与请求被提早取消的问题。

负责开发新服务的工程团队在确保`authzd`里包括MySQL查询执行的所有方法中都传入`Context`对象做的非常出色。将`Context`传入所有这些方法是必要的，因为这些方法是整个请求处理中最重要的部分。我们必须保证`database/sql`的查询方法有接收到我们请求的`Context`以确保我们可以在查询花费过长时间时讲查询取消。

然后在实践中，`authzd`似乎没有接收/执行取消（超时依旧发生）：

{% asset_img authzd_timeout.png 图3:authzd的响应时间。解析器的超时设置为1000毫秒（图中的红线），我们依旧可以看到许多5000毫秒的尖峰。%}

仅仅通过观察这些指标，我们可以很明显地得知虽然我们将请求的`Context`发送到了应用代码里，取消操作依旧被无视了。通过代码审查来寻找问题是非常费力的，尽管我们强烈怀疑超时问题是由Go MySQL驱动程序所引起的，因为它是我们服务中唯一一个执行对外请求的部分。在这个情况下，我从生产环境里的某一个服务器上拿到了stack trace用以确定到底是哪一块代码出了问题。我们不停地寻找相关的stack trace，终于当我们看到一段关于`QueryContext`的调用的跟踪日志时，原因就变得很明显了：

```Go
 0  0x00000000007704cb in net.(*sysDialer).doDialTCP
    at /usr/local/go/src/net/tcpsock_posix.go:64
 1  0x000000000077041a in net.(*sysDialer).dialTCP
    at /usr/local/go/src/net/tcpsock_posix.go:61
 2  0x00000000007374de in net.(*sysDialer).dialSingle
    at /usr/local/go/src/net/dial.go:571
 3  0x0000000000736d03 in net.(*sysDialer).dialSerial
    at /usr/local/go/src/net/dial.go:539
 4  0x00000000007355ad in net.(*Dialer).DialContext
    at /usr/local/go/src/net/dial.go:417
 5  0x000000000073472c in net.(*Dialer).Dial
    at /usr/local/go/src/net/dial.go:340
 6  0x00000000008fe651 in github.com/github/authzd/vendor/github.com/go-sql-driver/mysql.MySQLDriver.Open
    at /home/vmg/src/gopath/src/github.com/github/authzd/vendor/github.com/go-sql-driver/mysql/driver.go:77
 7  0x000000000091f0ff in github.com/github/authzd/vendor/github.com/go-sql-driver/mysql.(*MySQLDriver).Open
    at <autogenerated>:1
 8  0x0000000000645c60 in database/sql.dsnConnector.Connect
    at /usr/local/go/src/database/sql/sql.go:636
 9  0x000000000065b10d in database/sql.(*dsnConnector).Connect
    at <autogenerated>:1
10  0x000000000064968f in database/sql.(*DB).conn
    at /usr/local/go/src/database/sql/sql.go:1176
11  0x000000000065313e in database/sql.(*Stmt).connStmt
    at /usr/local/go/src/database/sql/sql.go:2409
12  0x0000000000653a44 in database/sql.(*Stmt).QueryContext
    at /usr/local/go/src/database/sql/sql.go:2461
[...]
```

问题出在我们并没有将`Context`传入到我们以为它会去的地方。尽管我们在执行SQL查询时将`Context`传入给了`ContextQuery`函数，并且该`Context`对象的确被用于执行和SQL超时查询，我们并没有注意到`database/sql/driver`API中的一个特殊情况：当我们尝试在连接池为空时执行查询，我们需要调用`driver.Driver.Open()`来创建新的连接，但是这个创建方法并不接受`context.Context`！

上面的的stack跟踪清楚地显示了`Context`是如何在第（8）行停止传播以及调用 `(*MysqlDriver).Open()` 与DSN连接到数据库的。打开MySQL连接的操作并不便宜：它涉及建立TCP连接、SSL协商（如果我们使用 SSL）、执行MySQL握手和身份验证以及设置各种默认连接选项等等。综上，至少有6个网络相关的连接步骤没有遵守我们的超时规定，因为它们并不知道（没有接收）我们的`Context`对象。

那么我们该怎么办呢？我们尝试的第一件事就是在我们的DSN连接上设置超时值，它可以间接设置`(*MySQLDriver).Open`使用的的TCP超时数值。但这并不够，因为打开TCP通道只是初始化连接的第一步。剩下的步骤比如MySQL握手等依旧没有遵守任何超时设定，所以我们仍然会有超过全局1000毫秒超时的请求。

正确的修复涉及重构Go MySQL驱动程序的一大部分代码。该问题是在`Go 1.8`版本实现`QueryContext`和`ExecContext`这两个API时引入的。尽管这些API由于接收了`Context`对象可用于取消SQL查询，但`driver.Open`方法没有随之跟着采用`context.Context`参数。在`Go 1.10`时添加了两个补丁，它引入了一个新的接口`Connector`并且与驱动程序分开实现了。也因此，同时支持`driver.Driver`和 `driver.Connector`两个接口需要对所有SQL驱动程序的结构进行大改。由于问题的复杂性且缺少对旧的`Driver.Open`接口的认识，随意更改可能导致可用性问题，因此Go MySQL驱动程序从未使用新接口。

幸运的是，我有时间和耐心进行重构。在`authzd`中使用了包含了新的接口实现的Go MySQL驱动程序后，我们可以在stack跟踪里验证`QueryContext`被传入了创建新的MySQL连接的函数里。

```Go
 0  0x000000000076facb in net.(*sysDialer).doDialTCP
    at /usr/local/go/src/net/tcpsock_posix.go:64
 1  0x000000000076fa1a in net.(*sysDialer).dialTCP
    at /usr/local/go/src/net/tcpsock_posix.go:61
 2  0x0000000000736ade in net.(*sysDialer).dialSingle
    at /usr/local/go/src/net/dial.go:571
 3  0x0000000000736303 in net.(*sysDialer).dialSerial
    at /usr/local/go/src/net/dial.go:539
 4  0x0000000000734bad in net.(*Dialer).DialContext
    at /usr/local/go/src/net/dial.go:417
 5  0x00000000008fdf3e in github.com/github/authzd/vendor/github.com/go-sql-driver/mysql.(*connector).Connect
    at /home/vmg/src/gopath/src/github.com/github/authzd/vendor/github.com/go-sql-driver/mysql/connector.go:43
 6  0x00000000006491ef in database/sql.(*DB).conn
    at /usr/local/go/src/database/sql/sql.go:1176
 7  0x0000000000652c9e in database/sql.(*Stmt).connStmt
    at /usr/local/go/src/database/sql/sql.go:2409
 8  0x00000000006535a4 in database/sql.(*Stmt).QueryContext
    at /usr/local/go/src/database/sql/sql.go:2461
[...]
```

请注意现在`(*DB).conn`是如何直接调用我们驱动程序的接口并传递当前`Context`的。很明显地，我们可以看到所有请求都遵守了全局请求超时设置：

{% asset_img response_times.png 图4:解析器响应时间，超时设置为90毫秒。可以看出`Context`的传入因为99百分位数看起来更像是条形码而不是某些图。%}

## 小提示 （Production tips）

- 你不需要对`sql.(*DB)`的初始化进行任何更改就可以使用新的`sql.OpenDB`API。即使在`Go 1.10`中调用`sql.Open`，`database/sql`包也会检测到`Connector`接口并使用`Context`创建新连接。

- 然而，`sql.OpenDB`和`mysql.NewConnector`一起使用有几个好处，最显著的是可以在`mysql.Config`结构外配置MySQL集群的连接选项，不需要调整DSN。

- 不要在MySQL驱动上设置`?timeout=`（或者与之相等的`mysql.(Config).Timeout`）。为dial超时设置一个静态值是一个坏主意，因为它不会计算你当前的请求预算。

- 相反，确保你应用中的每一个SQL操作都是用了`QueryContext`/`ExecContext`接口。这样可以根据你当前请求的`Context`在不同的情况下，比如连接失败或者查询过慢等时候取消请求。

# 竞争 （The race）

最后是一个由于数据竞争而产生的非常严重的安全问题。我们已经讨论了很多关于`sql.(*DB)`如何只是一个有状态连接池的包装（wrapper）的问题，但有一个细节我们还没有讨论。在进行一个对数据库进行最常见的操作之一，通过`(*DB).Query`或`(*DB).QueryContext`执行SQL查询时，该操作会从连接池里“窃取”一条连接。

不像没有从数据库返回实际数据的`(*DB).Exec`调用，`(*DB).Query`可能返回一行或多行数据。为了在应用中读取这些数据，我们必须控制这些连接以让服务器将写入数据。这就是每次调用`Query`和`QueryContext`时返回的`sql.(*Rows)`的用途：它包装了我们从连接池中借用的连接，因此我们可以从中读取单个行。这就是为什么在处理完查询结果后调用`(*Rows).Close`是非常重要的，因为不这么做的话，被借用的连接永远不会回到连接池里。

从Go的第一个稳定版本开始，使用`database/sql`的SQL查询的流程一直是固定的。比如：

```Go
rows, err := db.Query("SELECT a, b FROM some_table")
if err != nil {
    return err
}
defer rows.Close()

for rows.Next() {
    var a, b string
    if err := rows.Scan(&a, &b); err != nil {
        return err
    }
    ...
}
```

这在实践中很简单:`(*DB).Query`返回一个`sql.(*Rows)`，一个从SQL池中借来执行查询的连接。对`(*Rows).Next`的调用将从该连接中继续读取结果行，而我们可以通过调用`(*Rows).Scan`读取具体的内容。当我们调用`(*Rows).Close`后，`Next()`将返回`false`且`Scan()`将返回错误。

而涉及底层驱动程序的实现更为复杂：驱动程序返回自己的`Rows`接口，而`sql.(*Rows)`则是该接口的包装。但是，作为优化，驱动程序的`Rows`并没有`Scan`方法，它只有一个`Next(dest []Value) error`的迭代器。这个迭代器背后的想法是为SQL查询中的每一列返回Value对象。调用`sql.(*Rows).Next`相当于调用`driver.Rows.Next`，保留一个指向驱动程序返回的`Value`的指针。当用户调用`sql.(*Rows).Scan`时，标准库将驱动程序的`Value`对象转换为用户作为参数传递给`Scan`的目标类型。这意味着驱动程序并不执行参数转换（Go标准库转换），且驱动可以从它的`Rows`迭代器返回借用的内存而不是分配新的`Value`，因为`Scan`方法无论如何都会转换内存。

你可能已经猜到了，我们正在讨论的安全问题正是由内存借用引起的。返回临时内存实际上是一个重要的优化，驱动contract中也鼓励这样做，由于它允许我们返回可以直接在连接缓冲区读取结果的指针，这个优化在MySQL中效果很好。SQL查询的原数据会直接传给`Scan`方法，之后`Scan`会将该数据解析为用户期望的类型，比如整型、布尔型或者字符串等。这样做是没有安全问题的，因为在用户执行`sql.(*Rows).Next`之前，我们不会覆盖或者改动我们的连接缓存区，因此不会有数据竞争的情况。

而当`Go 1.8`引进了`QueryContext` API之后，事情就不一样了。我们已经在这篇文章中详细介绍了为什么使用新的接口允许我们通过调用`db.QueryContext`来提早中断一个查询，而问题就在于当我们提早中断一个正在进行结果行（resulting rows）的扫描（scanning）时，驱动程序会覆盖底层的连接缓冲（connection buffer），导致`Scan`返还损坏的数据，这就产生了安全漏洞。

当我们理解了取消`Context`是意味着在客户端而非是在MySQL服务器取消SQL查询后，这结果就不会令人惊讶了。MySQL服务器会继续通过连接为我们提供查询结果，所以当我们想要重用已经取消了查询的连接，我们必须“清理（drain）”服务器发来的所有数据包。为了排空和丢去所有查询中的结果行（rows），我们就必须读取连接的缓存区里的所有数据包。并且由于取消可以和`Scan`同时发生，我们就需要覆盖用户正在读取的`Values`之后的内存。也因此，很高概率用户会得到损坏的数据。

这个竞争（race）的确让人觉得可怕。但最可怕的是，你无法判断数据竞争是否已经或者正在应用中发生了。在之前的`rows.Scan`的例子中，尽管我们有进行错误检查（error checking），但其实并没有可靠的方法告诉我们该SQL的查询是否因为`Context`过期而被取消了。如果在`rows.Scan`的调用中发生了`Context`取消，且数据（rows values）已经被扫描，这个情况下很可能已经有损坏的数据但是不会有错误提示，因为`rows.Scan`只会在函数启动（entering the function)时对这些行（rows）进行检查。因此，当竞争发生时，我们就可能得到损坏的数据却不知情（没有错误提示）。在下一次调用`rows.Scan`之前，我们并不会知道查询是否被取消，且因为`rows.Next()`也一样不会犯错错误而是会返回`false`，我们永远不会进行`rows.Scan`的调用（译者注：所以该错误被永远的隐藏了）。

判断SQL查询是否没有错误地完成或查询是否已提前取消的唯一方法是检查`rows.Close`的返回值。我们没有这样做，因为该方法是在defer中被调用的。尴尬地是，通过在GitHub上检索`rows.Close()`方法，可以看到几乎没有人在他们的代码里特地检查`rows.Close()`的返回值。其实在`Go 1.8`引入`QueryContext`之前，这样做是没有问题的，因为`rows.Scan`总是能捕获错误。即使我们在Go MySQL的驱动程序里修复了这个数据竞争的问题，我们依旧无法在不检查`rows.Close()`的返回值的情况下得知我们是否已经扫描了所有的原数据，所以我们只好通过更新我们的linters（译者注：通过linter来检查代码，保证我们对Close方法进行了检查）。

我能想到不同的方法来修复这个问题。最简单的就是复制（clone）`driver.Rows.Next`返回的内存，但这会非常慢。驱动代码之所以这么写就是因为不希望在调用`driver.Rows.Next`时继续分配内存，因为分配的内存会一直持续到用户调用`sql.(*Rows).Scan`。如果我们在`Next`中对内存进行分配，我们将为每一行（row）分配两次内存，这意味这个错误修复将导致明显的性能下降。Go MySQL的维护者和我都对这个方案不满意。另外一个相似的方案也被拒绝了：在调用`sql.(*Rows).Close`时截断连接缓存，因为这个方案也可能在每一次`Scan`时进行内存分配。这些被拒绝的方案导致了维护者之间的僵局，且这个严重的问题被拖了6个月。我个人认为这样的BUG被拖延了6个月而不修复是非常可怕的，所以我必须找到一个不会导致性能下降的新方法来解决我们生产环境里的问题。

我尝试的第一个方案是在不碰触底层连接缓冲区的情况下“清空”连接。如果我们不把数据写入缓存里，就不会有数据竞争的问题发生了。而这马上就变成了我们的噩梦且导致了“意大利面式代码”。MySQL可以发生许许多多我们必须排出的数据包，并且每一个数据包都在标头（header）中定义了不同的包的大小，在没有缓存的情况下解析这些标头是非常麻烦（not pretty）的。

在经过一番思索之后，我终于想到一个有简单又快速，并且最终被同意了的解决方案：双缓冲。在旧的计算机图形堆栈中，人们会直接在帧缓冲区（frame buffer）中写入每一个像素，同时显示器会以给定的刷新率读取像素。在屏幕对帧缓冲区进行同时的读和写时，会出现图形故障——我们通常称之为闪烁。闪烁从根本上说是一种可以亲眼看到的数据竞赛，我猜你会同意这是一个非常酷的概念。修复计算机图形中闪烁的最直接方法是分配两个帧缓冲区：屏幕将从一个缓冲区（前缓冲区）进行读取，而图形处理器对另一个缓冲区（后缓冲区）进行写入操作。当图形处理器渲染完某一帧时，我们会将后台缓冲区与前台缓冲区进行翻转（这是一个原子性的操作），因此屏幕永远读取到任何正在组合的帧。

{% asset_img double_buffering.gif 图5:Nintendo64图形堆栈中的双缓冲.如果这对马里奥来说足够好，对于MySQL驱动程序而言应该也足够了。%}

这听起来很像MySQL驱动程序上的连接缓冲区所发生的情况，那为什么不以同样的方式修复它呢？在我们遇到的问题里，当`driver.Rows`因为查询被提前取消而被关闭时，我们可以交替连接缓冲与后台缓冲，这样保证了我们在清理后台缓冲时用户可以调用前端缓冲的`sql.Rows.Scan`。下一次使用同一个连接进行SQL查询时，我们会从后台缓冲读取数据，直到所有的行（rows）都被关闭我们才会再一次翻转缓冲使其回到原本的位置。这个实现是很简单的，尽管它依旧分配了内存，但整个MySQL连接的生命周期内只有一次分配，因此分配的开销很容易被分摊。通过将后台缓冲的内存分配延迟到我们第一次需要将数据注入，我们可以进一步优化性能，因此当没有查询被提早取消时，不会有额外的内存分配。

在小心地指定了一系列基准测试来保证双缓存方案不会导致任何性能下降后，这个修复提交终于被合并了。不幸的是，我们无法检查之前有多少损坏数据的情况发生，也许已经发生了很多次，因为尽管这个问题难以找到，但要触发这个问题却非常容易。

## 小提示 （Production tips）

- 始终对`(*Rows).Close`的错误返回进行显式检查。这是检测SQL查询在扫描过程中是否被中断的唯一方法。但是，请不要删除代码中的`defer rows.Close()`的调用，因为这是在扫描期间发生`panic`或提前返回时关闭行（Rows）的唯一方法。多次调用`(*Rows).Close`总是安全的。

- 永远不要在`(*Rows).Scan`里使用`sql.RawBytes`。尽管Go MySQL驱动程序现在已经更可靠了，但如果查询提前取消，访问`RawBytes`可能依旧会因为其他SQL驱动而失败。如果发生这种情况，很可能会读到无效数据。扫描`[]byte`和`sql.RawBytes`的唯一区别是原始版本不会分配额外的内存。这个小小的改变可以避免在使用`context`的程序里避免潜在的数据竞争（译者注：原文为This tiny optimization isn’t worth the potential data race in a Context-aware application，个人理解为使用`[]byte`可以避免读取失败。）

# 总结 （Wrapping up）

使用新的编程语言编写并将应用部署到生产环境始终是一个挑战，尤其是GitHub这种规模的应用。我们花了很多年为Ruby调试了MySQL客户端来支持现有的吞吐量和复杂性，我们很高兴现在能为Go MySQL驱动程序做同样的事情。我们在第一次对Go编写的应用进行部署并修复的程序现在已经作为Go MySQL驱动程序`1.5.0`版本的一部分并被普遍使用，随着我们越来越多地使用Go，我们将继续为驱动程序贡献更多修复和功能。

感谢Go MySQL维护者@methane和@julienschmidt，感谢他们帮助审查和发布这些补丁！
