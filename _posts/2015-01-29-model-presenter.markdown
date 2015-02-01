---
layout: post
title: "给你的 Model 减减肥"
date: 2015-01-29 15:00:00
author: hbc
categories: web programming mvc
---

一般来说，MVC 架构里面的 model 层会承载最多的功能，例如：

- （最基本的）数据记录功能;
- 将储存的数据用其他形式展示处理;
- （假如你使用 [fat model][fat-model]，还会有）业务逻辑封装功能。


假如我们将所有的实现都放到模型文件里面，那么就会出现 obese model 的情况。
为了减少因为功能过于集中带来的问题，我们可以对模型进行分拆，
以减少一个文件的代码行数（好 naive 的指标，但我就是那么容易满足 <(▰˘◡˘▰)> ）。


笼统地说，fat model 可以根据功能进行如下划分：

- 基本数据定义功能：保留在原模型定义中
- 视图性的功能：将其分拆到 presenter 之上（适用 decorator 模式）
- 逻辑封装的功能：将逻辑实现进行拆分，使用组合等方式进行解耦


下面以 laravel 的 eloquent 模型（why php?!）为例子介绍下这几种划分方法。


## 基本数据定义功能

假定我们现在有一个多用户的博客系统，那么这个系统里面的文章模型可以定义如下：

<script src="https://gist.github.com/bcho/5e6f791469b443b7707a.js"></script>


## 视图功能

视图功能一般用作将模型数据转换成 view 层次需要的结构（例如序列化成 JSON 格式）；
在 model 层完成的原因主要是：

1. 希望保持 controller 的苗条
1. 希望尽可能复用这些转换实现


从上面几个要求来看，我们需要做的就是在原来的模型上添加功能 --
所以十分适合用 decorator 模式来实现。

<script src="https://gist.github.com/bcho/da1fc4908f4d2e6f3980.js"></script>


## 逻辑封装功能

逻辑功能基本上都会和对应模型有很高的耦合度，所以视图功能中用到的 decorator 模式可能不太适合处理这种情形；
但不难发现，特定的业务逻辑基本上都只会在某几个方法中进行组织实现，
因此我们可以使用 composition 的方式来拆分这些逻辑块 (a.k.a `mixin` 或 `trait`)。

<script src="https://gist.github.com/bcho/3095ce2a426a2dbd6809.js"></script>

如果嫌 composition 这种方法还是不能解决耦合度高的问题，也可以尝试使用 observer
模式来进行分离，但这种方案主要适用在 [pub/sub 事件][eloquent-event]中。


## Recap

通过上述三种划分方法，我们可以将庞大的模型按照层次分拆成若干个不同的模块，
并且可以按需使用某一个模块；这样做既能降低模型的复杂度，也更容易进行测试。


[fat-model]: http://127.0.0.1:4000/2014/10/fat-model-and-service-layer.html
[eloquent-event]: http://laravel.com/docs/4.2/eloquent#model-events
