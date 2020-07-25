---
title: 理解CORS(1)
date: 2020-07-19 11:15:31
tags: 安全性
---

一年前在另外一个团队做Gateway API的时候，曾经因为要满足SOC2而研究过这个主题，但并没有好好记录下来。一年后不幸地被抓去做一个前端项目的PoC，又需要使用相关知识，于是记录一番。

## 1. Definition of an origin（同源的定义）
在讨论跨域之前，我们首先要理解什么是“同源”，即same origin。首先，一个URL由以下几个部分组成：<br/>
`[schema]://[host]:[port]`

以下为几个例子，方便理解：
我们以`http://account.example.com/home/index.html`为对象进行对比。

| URL    | 是否同源 | 备注 |
| :---- | :----:  | :---- |
| `http://account.example.com/settings/one.html` | 是 | Path并不属于同源比较对象。他们的schema，host与port完全温和所以不存在不同源的问题 |
| `http://account.example.com/logout.html` | 是 | Path并不属于同源比较对象。他们的schema，host与port完全温和所以不存在不同源的问题 |
| `https://account.example.com/logout.html` | 否 | <b>Schema</b>不同，一个是https一个是http |
| `http://profile.example.com/logout.html` | 否 | <b>Host</b>不同，一个是profile一个是account |
| `http://account.example.com:1000/logout.html` | 否 | <b>Port</b>不同，在http里面，默认的port是80，这个例子里是1000，所以不同源 |

<br/>
<br/>

## 2. 跨域资源共享（CORS)
在介绍完同源的定义后，我们可以开始了解什么是同源策略（Same-origin Policy）以及什么是跨域资源共享（CORS，Cross-Orgin Resource Sharing）。

- <b>Same-origin policy</b>: 一个简单但重要的浏览器安全策略。即一个网站只可以从自己的服务器加载资源。这个策略可以结合一些其他的安全策略比如httpOnly用来防止一些危险比如CSRF攻击、XSS攻击。
- <b>CORS</b>: 一个用于处理跨域访问资源的标准、机制。Same-origin policy作为一个基本的保障用户安全的存在，它在一定程度上防止了恶意攻击，但是也限制了一些正常的访问。这就是为什么我们需要使用CORS来解决跨域问题。

> 但是需要注意的是，跨域嵌入是允许的。浏览器允许加载页面中嵌入的来自不同源的scripts（脚本），图片以及媒体文件。

<br/>
<br/>

## 3. 进行跨域资源共享的主体与请求
首先，我们要理解该过程里面的两个主体：浏览器与服务器。
在整个CORS通信里面，前端开发者不需要进行任何操作，所有的设置将有浏览器自动完成，比如添加request headers，发送pre-flight request等。所以对于前端的开发者而言，是否启用CORS，他们的代码都将是一样的。与此不同的是，服务器端必须进行恰当的设置，CORS通信才可以实现。

其次，我们需要知道，在CORS里面有两种请求：简单请求（Simple Request）以及复杂请求（Complex Request）。

<b>简单请求:</b>
```
(1) Http Method必须满足以下任意一种:
  1. HEAD
  2. GET
  3. POST

(2) Http headers里面包含了以下一个或者更多：
  1. Accept
  2. Accept-Language
  3. Content-Language
  4. Content-Type （但仅限于application/x-www-form-urlencoded, multipart/form-data, text/plain这三种)
```

<b>复杂请求：</b>
```
所有不满足简单请求条件的都属于复杂请求。
```

那么为什么浏览器需要区分这两种请求呢？原因是这两种请求的处理是不一样的：
- 所有的简单请求都不会触发CORS preflighted request
- 任何可以不使用JavaScript也能触发的请求都属于简单请求，比如FORM POST或者使用<herf>的请求都是简单请求。且AJAX的跨域设计提到只要form可以发的请求，Ajax就可以发，而form一直都可以发出跨域请求，所以为了兼容form的设计，简单请求和复杂请求的处理也有不同的考虑。

<br/>
<br/>

## 4. 跨域请求的流程

#### 简单请求

1. 当用户开启一个页面并发送一个Ajax请求，浏览器会自动帮忙在请求里面加上`Origin`这个header。
2. 服务器会检查该header，并确认是否应该同意让该站点获取自己的资源。假如允许的话，则返回资源，并且response里面包含了`Access-Control-Allow-Origin: <origin>`这个header。
3. 当浏览器收到response后，它会检查`Access-Control-Allow-Origin: <origin>`这个header并且确认该header是否与当前的源吻合。如果不是，则拦截该response，反之则通过。假如服务器返回的是`Access-Control-Allow-Origin: *`，浏览器则会自动通过该response。

{% asset_img CORS_Sequence_Simple_Request.png sipmle request CORS flow %}

假设网站`http://a.example.com`想要访问`http://b.anotherexample.com`的资源，在没有设置CORS的情况下，则`http://a.example.com`会得到报错。而如果我们在`http://b.anotherexample.com`的服务器上设置了"Access-Control-Allow-Origin: *"则可以成功获得资源。简而言之，使用`Origin`与`Access-Control-Allow-Origin`就能完成最简单的CORS访问。在上图里面，服务器返回的是`Access-Control-Allow-Origin: *`，这代表了该资源可以被任意外域访问，这也就是为什么假如看到`*`，浏览器可以不需要检查。但是假如服务器端返回的是`Access-Control-Allow-Origin: http://a.example`这就代表了除了`http://a.example`其他外域均不允许访问该资源，这就是为什么浏览器会进行检查并且如果origin不一致就会拦截。

#### 复杂请求

1. 发送preflight request进行执行预检测。首先浏览器会使用`OPTIONS`方法发起该请求，服务器会告知浏览器是否允许该实际请求。<b>Preflight request实际上是用于避免CORS请求对于服务器的用户数据可能产生的未知影响。</b>
2. 假如服务器同意了该请求，则浏览器会重复简单请求的流程，发送真正的请求。

{% asset_img CORS_Sequence_Complex_Request.png sipmle request CORS flow %}

从上图中，我们可以看出，我们将要发送的请求里面包含了一个自定义的header `X-PINGOTHER:pingpong`，于是需要进行预先检查：

| Header    | 解释 |
| :---- | :---- |
| `Access-Control-Request-Method: POST` | 实际请求将会是POST，是否同意？|
| `Access-Control-Request-Headers: X-PINGOTHER, Content-Type`| 实际请求包含了`X-PINGOTHER`与`Content-Type`，是否同意？|


服务器检查后，返回说：

| Header    | 解释 |
| :---- | :---- |
| `Access-Control-Request-Method: POST, GET, OPTIONS` | 可以使用`POST`，`GET`和`OPTIONS`发起请求 |
| `Access-Control-Request-Headers: X-PINGOTHER, Content-Type` | 允许携带这两个headers |
| `Access-Control-Max-Age: 86400` | 这个response的有效时间是86400秒，即24个小时。在24个小时内，相同的请求不需要再一次发送preflight request。这个有效时间将由浏览器进行管理，如果超过了最大时间，则实际请求不会继续有效。需要重新发送preflight request。 |


<br/>
<br/>





