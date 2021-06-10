---
title: 理解Vue的Render渲染函数
date: 2018-09-19 10:02:11
tags: [前端, VueJs]
---

<br/>
<br/>
<br/>
<br/>
最近在尝试将Vue的SSR与.Net Core的Js SSR Middleware结合，让纯客户端应用转变为服务端渲染。发现自己对Vue的render函数很不了解，所以打算好好整理一下，至少理解个大概。
<br/>
<br/>

## 1. HTML、节点与树（HTML, Nodes and Tree）

首先，Vue的[官方文档](https://cn.vuejs.org/v2/guide/render-function.html)里提供了关于浏览器的“[DOM Tree](https://javascript.info/dom-nodes)”工作原理。
> 每个元素都是一个节点。每片文字也是一个节点。甚至注释也都是节点。一个节点就是页面的一个部分。就像家谱树一样，每个节点都可以有孩子节点 (也就是说每个部分可以包含其它的一些部分)。

最中心的思想则是：
> if something’s in HTML, then it also must be in the DOM tree.

于是我们知道，对于浏览器而言，当它读到一段HTML代码，它会将这段HTML转成一个“DOM Tree（节点树）”。
<br/>
<br/>

## 2. 什么是render（渲染）函数呢？

Vue是通过 template（字符串模板）来创建HTML的。render函数与 template 十分相似，都是用来高效的更新所有这些节点的——准确的说，[render函数是字符串模板的代替方案，允许你发挥 JavaScript 最大的编程能力。](https://cn.vuejs.org/v2/api/#render)
<br/>
<br/>

## 3. 虚拟DOM（The Virtual DOM）

在了解了到底什么是render函数之后，下一步是理解这个函数如何工作的。
> 该渲染函数接收一个 createElement 方法作为第一个参数用来创建 VNode。如果组件是一个函数组件，渲染函数还会接收一个额外的 context 参数，为没有实例的函数组件提供上下文信息。

创建VNode，即Vue通过建立一个虚拟 DOM 对真实 DOM 发生的变化保持追踪。createElement 方法会返回一段信息，该信息会告诉 Vue 页面上需要渲染什么样的节点，及其子节点。而具体createElement 方法可能又需要另外一篇文章来解析了。又需要提及的是，Vue 的模板实际是编译成了 render 函数，一言以盖之，Vue利用render函数来创造真正的DOM Tree。
<br/>
<br/>

## 4. Vue 的渲染机制

所以，Vue的渲染机制大致上可以这么说：
template → render function → virutal DOM → real DOM
template会被编译成render funciton， render function会创建虚拟DOM，虚拟DOM最后影响创造真实的DOM Tree。

这里又可以提一下Vue的两个构建概念：<u>独立构建</u>与<u>运行时构建</u>。*
独立构建：包含模板编译器，渲染过程HTML字符串 → render函数 → VNode → 真实DOM节点
运行时构建：不包含模板编译器，渲染过程render函数 → VNode → 真实DOM节点
通过比较可以注意到，独立构建多了一个模板编译器。所以直接使用render函数可以做到省去编译器只使用运行时版本。
<br/>
<br/>

## 5. 总结

在读了官方文档、网上关于render function的文章以及稍作总结后，个人总结render函数其实是Vue 将HTML字符串，或者Js funciton、object转化成真正的DOM的一个中介。所以需要深入理解 createElement 到底可以接收什么参数以及 Virtual DOM 是如何作用的。

*做笔记的时候发现 createElement 和 VNode 是两个很核心的知识点。应该需要用另外了解。
<br/>
<br/>
<br/>
<br/>

参考文章
- [Vue官方文档—Render Functions & JSX](https://vuejs.org/v2/guide/render-function.html)
- [Vue 2.0学习笔记：Vue的render函数](https://www.w3cplus.com/vue/vue-render-function.html)
- [Vue原理解析之Virtual Dom](https://segmentfault.com/a/1190000008291645)