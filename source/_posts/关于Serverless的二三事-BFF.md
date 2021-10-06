---
title: 关于Serverless的二三事
date: 2020-07-20 14:15:32
tags: [前端, 架构]
---
<br/>
<br/>

因为被丢去写前端了嘛，必须使用一个公司内部的框架。研究了一阵子后发现有一个serverless的概念属于以前听说过，大致明白含义但是从来没有好好了解。决定开一个系列看看能不能讲明白。
在开始讲serverless之前，先谈一谈什么是Backend-for-Frontend(BFF)。



## 1. 什么是Backend-for-Frontend（BFF)
在现有的结构里面，大部分的设计都是前端直接和后端API进行通信。在这种模式下，如果前端需要一些改动或者增加新的feature，就需要后端团队进行配合，至少得把API的接口设计好才可以继续进行开发。那么弊端就会出现了：
- 资源调度问题：没有足够的人手进行后端开发
- 需求问题：该API也许是用于多平台多应用的开发，即别的平台/应用可能没有改动这个API的需求。不同的应用有不同的需求，该API可能无法满足所有的需求。从而需要进行“general”化。
- 管理问题：前端和后端团队的任务优先级不同等等
这些因素都可以导致开发周期延长，维护成本增加以及其他问题，于是就有了BFF，自如其名，即，专门为了前端而存在的“后端”或者说一个“中间件”。

### 传统意义上的前后端架构：
{% asset_img backend-frontend.png backend-frontend %}

### BFF架构：
{% asset_img backend-for-frontend-example.png backend-for-frontend-example %}

<br/>

可以很明显的看出，BFF pattern把一个很复杂的大型后端分解成了多个BFF。BFF设计模式旨在为所有API保留一个接口，并充当前端和API之间的转换层。它大部分时候是用来作为一个“转接器”，它会汇总所有downstream services的数据，并根据当前的前端设计进行数据转换。BFF本身并不实现业务应用逻辑，它仅仅是转换（transformation）和聚合（aggregation）。由于BFF是面向用户体验的，往往是由实现该部分UI的前端团队负责实现以及维护，即UI与BFF由同一个团队负责。前端团队往往更理解哪些数据是必要的，所以可以更好的理解需求以及实现数据转换。

一言以蔽之，BFF是一个解除后端API和前端UI耦合的设计。Decouple the backend API and frontend UI.

<br/>
<br/>

## 2. BFF的架构

在BFF里面，有三个主体：
- Backend： 后端。负责数据模型（data model），数据库交互等等。
- BFF: 与后端交流，根据UI来定义或者调整API，简化Client Side（客户端）和Server Side（服务端）的发布流程，且负责提供一个API集合给对应的前端。
- Frontend： 前端。负责UI。

目前主要有两种架构BFF的方式：

1. 每一个前端团队都有自己的BFF。
2. 所有的前端团队一起分享同一个BFF。

### 每一个前端团队都有自己的BFF
这种架构方式细化了每一个平台的需求。后端可以放心地提供信息，BFF部分则负责优化，转换。

{% asset_img backend_frontend_oneBFF.png backend frontend one BFF %}

### 多个前端团队拥有同一个BFF
这种架构方式可以提高开发效率，因为所有的前端API需求都被放在一起集中开发。其实这种情况下，BFF就是一个API Gateway，它负责了对其他downstream service的对接。

{% asset_img backend-for-frontend-example.png backend-for-frontend-example %}

<br/>
<br/>

## 3. BFF的具体实现与问题

关于BFF的实现，有三个问题要考虑：
- 拆分程度
- 技术栈管理
- 复用问题

### 拆分程度

上面以及聊过两种不同的架构，那么在实现上，又有什么考虑呢？首先是拆分程度。既然我们要将一个大后端拆分成多个BFF，那么要根据什么来拆分呢：
- UX/UI：根据用户的交互体验。比如PC端的体验和移动端的体验不同，可以考虑拆分出2个BFF。
- 端级：根据不同的设备来进行拆分。比如iOS与Android同属于移动端，但是他们隶属于不同的前端设备，我们也可以根据这个来拆分。
- 团队：根据不同的团队来拆分。即每一个前端团队都拥有一个自己的BFF。

通常情况下，我们会选择UX/UI来拆分。团队拆分很大程度上等同于UX/UI（但是还是看每个公司的组织结构），而端级的拆分很可能导致大量的代码重复，因为接口总是相同的。

### 技术栈问题

Micro-Service已然是行业的一个趋势，但是依赖问题始终令人头疼。在同一个生态下面的不同服务往往都有自己的通信和工作方式，常常需要依赖专门设计的接口进行通信。这种情况下，我们往往需要考虑：
- 即如何对接不同的downstream service：对于不同的接口，我们是应该使用RPC协议，Rest还是GraphQL？
- 如何管理组合这些API：该如何合理进行异步的调用，如何引用事件机制（比如RX）将这些异步的控制流程进行简化？
- 当某一个API调用失败时，该如何应对：是否应该在BFF层引入容错？如何设计熔断、重试机制才能合理的保护整个系统？是否需要提供不完整的响应内容？
- 等等……

### 复用问题

进行拆分就意味着重复逻辑的出现。一些通用逻辑比如：authentication，rate limit（限流）等等会导致大量的重复代码，并且重复代码就意味着我们难以保证统一更新以及使用起来有更高的学习曲线。由于我们是为了灵活性才拆分出多个BFF，这种情况下我们不可能再次将这些BFF合并。于是就需要多一个中间层进行通用逻辑的拆分：
- 公共库
- 独立服务
- downstream service（下游服务，依赖树中的一环）
问题又来了……抽取重复部分就会导致耦合……不管是什么解决方案，前提是<b>技术栈相同</b>以及<b>该冗余是不可以忍受的</b>。

比较好的一个方案是直接使用网关，使用网关来进行一些基础功能比如routing，authentication&authorization，monitor，rate limiting等等……

<br/>
<br/>

## 4. 业界实践

{% asset_img BFF-implementation.png %}

<br/>
<br/>

## 5. 参考资料
1. [微服务架构~BFF和网关是如何演化出来的](https://www.cnblogs.com/alimayun/p/12063401.html) 
2. [Why big companies and rapidly growing startups need Back-end for Front-end](https://medium.com/blue-harvest-tech-blog/why-big-companies-and-rapidly-growing-startups-need-back-end-for-front-end-ee8e6ab8f575) 
3. [Backend For Frontend (BFF)](https://cloud.tencent.com/developer/article/1444653) 
4. [Building a Backend for Frontend (BFF) For Your Micro-Services](https://nordicapis.com/building-a-backend-for-frontend-shim-for-your-microservices/) 
5. [Backends for Frontends pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends) 