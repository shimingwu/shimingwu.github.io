---
title: 理解OAuth2(1)
date: 2020-07-09 16:59:27
tags: 
---
<br/>
<br/>

两三年前在创业公司的时候，曾经使用过OAuth2，虽然对流程有着大致的理解，但是很多词汇以及术语总是记不清楚。这次的项目又一次需要用到这方面的知识，于是打算好好记一下，也顺便整理思路，希望能把东西讲明白。

## 1. OAuth2

OAuth2是目前较为流行的authorization机制，用来授权第三方应用，获得用户数据。

<br/>
<br/>

## 2. Authorization and Authentication

最开始，我们先需要理解的是两个概念：
- <b>Authentication:</b> 
  Validating a user is the person he or she claims to be.
  确认用户。证明自己是该用户，比如提供密码来证明我的确是该账号的使用者。
- <b>Authorization：</b>
  Validating what permission the user has. 
  确认用户权限。当一个用户证明了自己可以使用某应用后，他可以对该应用做什么。比如是普通用户，还是管理员等。

打个比方，小明在公司是数据库管理员，小王在公司是财务。小明和小王都可以刷卡进公司系统，只需要他们证明自己是小明/小王即可，这就是authentication。
但是小明没有进入公司财务系统的权限，因为他不是财务。同样，小王也没有进入数据库的权限。这个就是他们的permission。所以只有在小明获得/展示他的数据库管理员的身份/职位的时候，他才会获得数据库的权限。这就是authorization。

<br/>
<br/>

## 3. OAuth2.0 专业术语

- <b>Resource Owner:</b> 即用户，数据的拥有者。
- <b>Client:</b> 用户想要使用的应用、网站等。
- <b>Authorization Server:</b> 用户用来登录账户，提供权限的服务器。有的时候Authorization Server和Resource Server可以是同一个服务器。即该服务器既可以用来验证登录，也同时提供API服务。
- <b>Resource Server:</b> 存储用户数据或者提供用户数据的API的服务器。
- <b>Authorization Grant:</b> 这个Grant是一个凭证，它证明了用户同意该应用使用她/他的账户信息。
- <b>Redirect URI:</b> 当用户同意登录后，跳转回该URI。
- <b>Access Token:</b> 这个令牌可以证明请求是有效的。应用可以使用该令牌去Resource Server请求一些必需的用户资料（用户在Authorization Server已经同意了给予该权限）。
- <b>Scope:</b> 定义了该应用可以获得的用户资源。比如可以获得头像信息，邮箱信息或者全部信息等等。

一个很简单的场景。我想在lofter.com上发表文章，但是我不想在lofter上面注册lofter专用的账号了，我希望使用我的google账号来登录lofter.com并发表文章，于是我们有的下面的流程:
用户（<u>Resource Owner</u>）在浏览器输入"lofter.com"进入lofter网站（<u>Client</u>），点击“登录”，网站会跳转到"accounts.google.com"（<u>Authorization Server</u>)，google的登录界面出现，并显示出lofter想要使用的用户权限，比如获取头像信息，用户名（这些就是<u>scope</u>）等等，当用户同意，根据lofter的设置，用户会被带回lofter的<u>Redirect URI</u>且此时lofter获得了用户通过google颁发的凭证（<u>Authorization Grant</u>），通过这个凭证， lofter会再一次与"accounts.google.com"对话，交换<u>Access Token</u>，接下来，凭借着该<u>Access Token</u>，lofter就可以与<u>Resource Server</u>通信并获得头像信息以及用户名了。

<br/>
<br/>

## 4. Grant Type （授权模式）

在上面的场景中，我们注意到Authorization Grant的存在，即凭证、授权。并且，我们需要使用该凭证来换取令牌（Access Token）。这是为什么呢？
因为现实生活里面，不同的应用有不同的设计，有的网站可能是纯单界面应用（Single Page Application），或者Serverless，又或者是手机应用，它们有的只有前端没有服务器，有的有自己的服务器，这样的情况下，就导致了流程里某些需要使用服务器来储存获得安全性的设计无法实现。于是OAuth2.0里面就有了不同的Grant Type，来确保不同的应用可以选择适合自己的验证流程。

目前常用的OAuth Grant Types有：
- <b>Authorization Code （授权码模式）:</b> 
功能最完整，流程最严密的授权模式。在authorization server真正提供access token之前，client必须获得authorization code，获得了code后，client再次向authorization server进行access token exchange，并且该步骤是在后台服务器上执行的，对用户完全不可见，因此获得了一定的安全性。
- <b>Implicit Grant（简化模式）：</b> 
该模式下，申请access token并不需要通过后台服务器，因此属于authorization code的简化版本。当用户在authorization server上同意了授权后，返回redirect URI的时候，URI里面的Hash部分已经包含了access token（从而省略了使用authorization code交换token这个步骤）。
- <b>Client Credentials</b> 
- <b>Resource Owner Password Credentials</b> 

<br/>
<br/>

## 5. OAuth2.0 Authorization Code Flow

下图为OAuth2.0 authorization code的简易流程。

{% asset_img oauth2_authorization_code_flow.png This shows the authorization code flow %}

<br/>
<br/>
