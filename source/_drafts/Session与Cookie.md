---
title: Session与Cookie
tags: 安全性
---

<br/>
<br/>
这个概念在前一家公司做全栈的时候就云里雾里，没有很明白。这次有幸在大公司做简单的架构，需要好好弄清楚一下。

## 1. 为什么需要Cookie/Session机制

当我们在网上搜索这个问题的时候，往往看到的第一句话就是：

> HTTP是一个无状态的协议。

在开始理解cookie和session之前，先来聊一聊这句话的含义：
1. 每一次的请求都是独立的。服务器不会知道该请求是否与其他的请求有关联，只要该请求是合法的，就会处理。
2. 服务器也不认识任何客户端，不管是否之前发送过合法的请求，也不管发送过多少次。
简而言之，<b>服务器不会记住任何事情</b>。

打个比方，小王开了一家餐馆，小明去餐馆吃饭，两人相谈甚欢，恨不得相约下一次再叙：
- Stateless（无状态）：如果是stateless，小明下次再去，小王就会不记得这件事，小明该付钱还是付钱，该重新介绍自己还是得重新介绍自己。
- Stateful（有状态）：如果是stateful，小明下次再去，小王一看，热情相拥，说终于来了，甚至有可能给小明打折。

这就是是否有状态的区别。于是问题来了，假如客户端和服务器请求资源的时候需要提供凭证，比如用户名和密码，由于HTTP是无状态的协议，用户每一次请求都需要重新输入自己的密码，这个设计就很不合理。于是，cookie/session作为状态管理的机制就应运而生了。

<br/>
<br/>

## 2. 什么是Cookie和Session

我们已经了解过，Cookie和Session是为了维护会话状态而产生的机制，它们可以让服务器知道自己现在是在和谁通话，该用户是否登录等等。

### Cookie
那么Cookie到底是什么呢？Cookie其实只是一段保存在浏览器的文件，它的大小不会超过4KB，其中会包含不同的key-value pairs（键值对）。Cookie有两种保存方式：保存在内存里的叫做 `session cookie` ，保存在硬盘里的，叫做 `persistent cookie`。`session cookie` 由浏览器维护，当用户关掉浏览器后就会被删除。`persistent cookie` 则与之相反，它有固定的过期时间，只有到了该过期时间或者用户手动清理，不然这个cookie会一直存在。

### Session





## 3. Cookie和Session的区别

## 4. Cookie的实现

## 5. Session的实现



https://en.wikipedia.org/wiki/HTTP_cookie
https://www.zhihu.com/question/19786827
https://www.zhihu.com/question/23202402
https://juejin.im/entry/6844903434366222350
https://stackoverflow.com/questions/32563236/relation-between-sessions-and-cookies
https://blog.csdn.net/duan1078774504/article/details/51912868
https://ruby-china.org/topics/33313
https://www.guru99.com/difference-between-cookie-session.html
https://medium.com/@rawat.hemant27/what-is-the-difference-between-cookie-cache-and-session-d6f468a9b4b3
https://juejin.im/post/6844903952157245453
https://www.bigcommerce.com/ecommerce-answers/what-cookie-and-why-it-important/
https://zh.wikipedia.org/wiki/Cookie