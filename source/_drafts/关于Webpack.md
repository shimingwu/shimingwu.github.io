---
title: 关于Webpack
date: 2020-07-21 21:00:54
tags: 前端
---

谁能想到两年前做的东西……两年后要重温一遍呢。

## 1. 什么是Webpack
简单来说，webpack是一个<u>静态模块打包工具</u>。它的主要概念就是将多个JavaScript，css，image等不同的任何类型的文件捆绑在一起。它会分析这些文件之间的依赖关系，比如什么文件import（导入）了什么文件，并且在内部生成一个dependency graph（依赖图），从而生成一个或者多个bundle（捆绑）文件。
但webpack又不仅仅只是一个捆绑工具，它同时也可以帮忙处理优化文件，如manifest；也可以使用loader去进行一些转义工作，使得不同类型的文件可以被捆绑；还可以使用plugin去进行environment variable injection等操作。但所有这些处理的最终目的都是为了将不同的文件处理成适合的浏览器可以接受的格式并且打包。

## 2. Webpack如何工作
