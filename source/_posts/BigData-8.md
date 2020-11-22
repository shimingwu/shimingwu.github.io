---
title: BigData(8)
tags:
  - Big Data
date: 2020-11-12 17:51:50
---



This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Data Stream
Data stream is a sequence of items. There are two problems with data stream:

- We are not able to store all data
- We are not able to exam every single item

There are multiple challenges faced by stream processing:

- The data itself is high volume, arrival speed is fast
- Machine only has limited memory and storage
- The latency requirement is strict
- Load balancing
- The message delivery is unreliable
- Fault tolerance
- ...

## General Strategies

- `Sampling`: Reduce the number of data items
- `Hashing`: Reduce computation overhead and/or data size

## Reservoir Sampling
Usually, if we want to take samples, we need to know all N data points then take the samples. However, for streaming, we might not be able to know what N is. Thus, in the data stream, we need to make a decision to keep a sample or not.

### K = 1
When k = 1, which means only 1 sample is picked for the whole set of `N` data items, we will have, the probability of each item be sampled should be `1/N`.
Thus, what we can do is:

- When 1st item n1 arrives, keep it. p(n1) = 1/1 = 1
- When 2nd item n2 arrives, the probability to keep it is `1/2`, then the probability of taking n1 is: p(n1)= 1 * (1-1/2) = 1/2, and for n2 is p(n2) = 1/2
- When 3rd item n3 arrives, the probability to keep it is `1/3`, then the probability of taking n1 is: p(n1) = 1 * (1-1/2) * (1-1/3) = 1/3. For n2, it is p(n2) = 1/2 * (1-1/3) = 1/3, for n3 is p(n3) = 1/3
- ...
- Thus, for item `i`, we have p(n1) = p(n2) = .. p(ni-1) = (1/i-1) * (1-1/i) = 1/i, and p(ni) = 1/i

### K > 1
When k > 1, which means multiple samples are picked for the whole set of `N` data items, we will have, the probability of each item be sampled should be `k/N`.
Thus, what we can do is:

- For the first `k` items, the p(n1) = p(n2) = ... = p(nk) = 1
- For `k+1`th item, the probability to keep it is `k/(k+1)`, and the probability to keep the item `r` inside the first `k` items is:
p(nr) = p(item r was kept in last round) * p(item n+1 is not kept) + p(item n+1 is kept) * p(item r was not replaced by n+1)
= `(1/k+1) + (k/k+1)*(k-1)/k = k/k+1`
- ...
- Thus, for item i (i>k), the probability to keep it is `k/i`


## Flajolet-Martin(FM) counter

What a FM counter can do? Assume we have 100 billion numbers, how can we find the number of unique numbers out of the 100 billion numbers?
>假设n个object，其中有m个唯一的，那么FM算法只需要log(m)的内存占用（实际操作中会是k*log(m)），以及O(n)的运算时间。当然，FM的问题是，它的结果只是一个估计值，不是精确结果。

## Bloom Filter
- If the result is not found, the element does not exist
- If the result is found, the element **may** exist

## Count-Min Sketches (CMS)
> Count-Min Sketch 是数据库中用到的一种 Sketch，所谓 sketch 就是用很少的一点数据来描述全体数据的特性，牺牲了准确性但是代价变得很低。

> 问题：统计一个实时的数据流中元素出现的频率，并且准备随时回答某个元素出现的频率，不需要的精确的计数，那该怎么办？


## Storm

> Apache Storm是一个分布式的、可靠的、容错的实时数据流处理框架。它与Spark Streaming的最大区别在于它是逐个处理流式数据事件，而Spark Streaming是微批次处理，因此，它比Spark Streaming更实时。

Nimbus：即Storm的Master，负责资源分配和任务调度。一个Storm集群只有一个Nimbus。

Supervisor：即Storm的Slave，负责接收Nimbus分配的任务，管理所有Worker，一个Supervisor节点中包含多个Worker进程。

Worker：工作进程，每个工作进程中都有多个Task。

Task：任务，在 Storm 集群中每个 Spout 和 Bolt 都由若干个任务（tasks）来执行。每个任务都与一个执行线程相对应。

Topology：计算拓扑，Storm 的拓扑是对实时计算应用逻辑的封装，它的作用与 MapReduce 的任务（Job）很相似，区别在于 MapReduce 的一个 Job 在得到结果之后总会结束，而拓扑会一直在集群中运行，直到你手动去终止它。拓扑还可以理解成由一系列通过数据流（Stream Grouping）相互关联的 Spout 和 Bolt 组成的的拓扑结构。

Stream：数据流（Streams）是 Storm 中最核心的抽象概念。一个数据流指的是在分布式环境中并行创建、处理的一组元组（tuple）的无界序列。数据流可以由一种能够表述数据流中元组的域（fields）的模式来定义。

Spout：数据源（Spout）是拓扑中数据流的来源。一般 Spout 会从一个外部的数据源读取元组然后将他们发送到拓扑中。根据需求的不同，Spout 既可以定义为可靠的数据源，也可以定义为不可靠的数据源。一个可靠的 Spout 能够在它发送的元组处理失败时重新发送该元组，以确保所有的元组都能得到正确的处理；相对应的，不可靠的 Spout 就不会在元组发送之后对元组进行任何其他的处理。一个 Spout 可以发送多个数据流。

Bolt：拓扑中所有的数据处理均是由 Bolt 完成的。通过数据过滤（filtering）、函数处理（functions）、聚合（aggregations）、联结（joins）、数据库交互等功能，Bolt 几乎能够完成任何一种数据处理需求。一个 Bolt 可以实现简单的数据流转换，而更复杂的数据流变换通常需要使用多个 Bolt 并通过多个步骤完成。

Stream grouping：为拓扑中的每个 Bolt 的确定输入数据流是定义一个拓扑的重要环节。数据流分组定义了在 Bolt 的不同任务（tasks）中划分数据流的方式。在 Storm 中有八种内置的数据流分组方式。

Reliability：可靠性。Storm 可以通过拓扑来确保每个发送的元组都能得到正确处理。通过跟踪由 Spout 发出的每个元组构成的元组树可以确定元组是否已经完成处理。每个拓扑都有一个“消息延时”参数，如果 Storm 在延时时间内没有检测到元组是否处理完成，就会将该元组标记为处理失败，并会在稍后重新发送该元组。


<br/>
<br/>

## Reference
- [水塘抽样（Reservoir Sampling）](https://zhuanlan.zhihu.com/p/29178293)
- [什么是水塘抽样算法（Reservoir Sampling）](https://cloud.tencent.com/developer/article/1376934)
- [Flajolet-Martin算法](https://greatpowerlaw.wordpress.com/2012/10/14/flajoletmartin/)
- [Flajolet-Martin算法](https://blog.csdn.net/baimafujinji/article/details/6472658)
- [Count-Min Sketch](https://florian.github.io/count-min-sketch/)
- [Count-Min Sketch 算法，解决大数据统计难题](https://mp.weixin.qq.com/s?__biz=MjM5ODIzNDQ3Mw==&mid=2649967190&idx=1&sn=67406c916f75c3f2ee59ce05d3f65ea8&chksm=beca3a5089bdb34652a7dbf7b399379f8ccfa3f941c1752dec529b1532018189cb0e4edb12b8&mpshare=1&scene=1&srcid=0622k0SqC0P2ryonJeaM5bVR&pass_ticket=4R9ZWXz%2FDcFGK9tVfysrmusZ3J09OLC1S3R3B%2BhZmqJ6QAiynVx5CyF%2By1Z2dT4Y#rd)


