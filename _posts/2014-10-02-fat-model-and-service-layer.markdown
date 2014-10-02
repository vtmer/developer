---
layout: post
title: "Fat Model 和 Service Layer"
date: 2014-10-02 21:25:00
author: hbc
categories: web programming mvc
---

基本上能看到的 [web framework](http://en.wikipedia.org/wiki/Comparison_of_web_application_frameworks) 都会标榜自己完美支持 MVC 这个模式；而大部分新手也会直接跳过了从零开始搭脚手架的过程，
直接跟着框架的设计走来实现需要的功能。


当然，基本每一个新任驾驶员摸爬滚打的过程都是做一个博客之类的应用，所以完全可以跟着框架本身的脉络来实现自己想要的功能。
但是，在驾驶员不断积累经验之后，大家都会发现其实大部分 serious 的项目都不能直接**只**用“官方教程”里面的套路，
而需要自己去进行一些“微创新”。接下来介绍的是两种比较流行的各有特色的架构方式。


## MVC

当然咯，在开始介绍这两种不同架构之前先让我们复习一下什么是 MVC，这样会更加方便我们进行交叉对比。

MVC 具体架构可以直接从名字看出（其实我第一次遇到这个缩写的时候没看出来 ~\_~）：

- **M** for model, 模型层，提供的是对数据**状态**的描述
- **V** for view, 视图层，提供到用户的界面显示功能
- **C** for controller, 控制器层，可以连接模型层和视图层，以为两者传送消息


当然在日常的 web 开发中，大部分框架都会直接把 model 做成到持久性储存（99% 的情况下指的都是数据库）的入口；
controller 将会负责最主要的业务规则（businese rules），最后让 view 来处理页面渲染的问题。


为什么大部分 web framework 都采用 mvc 的架构而不是其他的模式？
一个比较重要的原因就是通过将实际需求（基本上就是用户浏览你的数据）分拆成耦合度很小的 3 个部分：

- 数据持久化 (M)
- 数据处理 (C)
- 数据打包 (V)

这么一来，项目代码将会变得更加**模块化**，同一个项目内大家写的代码骨架都可以做到看起来差不多了。


## Fat Model

首先设想这么一个场景，我们现在写的博客需要既在文章内页显示评论，又要在首页显示所有文章的最新评论；
那么在传统的 MVC 里面我们可能会这么处理：

- 在 `PostController` 里面获取对应文章、对应文章的评论，显示它们；
- 在 `IndexController` 里面获取所有文章、所有文章的评论并按时间排序，显示它们。


显而易见这里就出现了一个 [smell point](http://en.wikipedia.org/wiki/Code_smell#Common_code_smells)：
在两个 controller 里面都出现了类似的操作：获取某个文章的评论。


作为有经验的驾驶员，我们会立刻想到通过复用代码的方法来解决“重复代码”这个问题 —— 但这个复用后的代码应该放哪里呢？
这时候，Fat Model 就会告诉你：

![Put Them In The Model](http://i.imgur.com/kTzdgcC.png)


Fat Model 实际上就是将部分本来在 controller 里面完成的逻辑放到**模型**的**方法**里面完成。
把模型从记录状态的角色提升到数据加工者 —— 这样一来，我们就可以在不同的 controller 里面复用这些重复的代码了。


不过当你选择抽离那些逻辑的时候可能会有些犹豫 —— 究竟这个获取文章列表的操作要不要提公因式呢？
那么这个对评论分页的操作呢？


其实只要我们细加区分，不难发现 controller 里要处理的逻辑不外乎两种：

- 和 view 相关的（例如分页操作、转换一下数据格式）
- 和 model 相关的（例如按某些特性要求去获取记录）

聪明的你绝对会抢着告诉我那种和 model 相关的逻辑都可以放到 model 的方法里面去。
没错，大部分情况下都是这样的 —— 我们可以直接给这种逻辑一个具体的名称：**domain logic** （领域特定逻辑）
这种逻辑一个最大的特点就是和具体实现的业务切合，而且不同模块之间复用很多。


所以在 fat model 模式下，看到 domain logic 请勇敢地把它们放到模型的方法里面去!

#### FYI

[fat model, skinny controller](http://weblog.jamisbuck.org/2006/10/18/skinny-controller-fat-model) 是 rails on ruby 中一种比较流行的写法。

-----------------------------------

## Service Layer

但是不是所有类型的 web 服务都适合用 MVC/Fat Model 的模式来开发呢？如果是这样的话，
我们还要架构师来干什么呢？显然传统的 MVC、 Fat Model 模式在处理负责的 domain logic 上会有比较大的问题。


考虑一个 ERP 系统，我们要对实体产品进行跟踪管理，这个过程涉及以下几个 domain logic：

- 库存管理
- 销售管理
- 物流管理


OK，现在我们要开始对系统进行建模 —— 那么针对商品这个模型来说，是不是需要把所有的逻辑都扔到模型方法上呢？

![Too Many Lines](http://i.imgur.com/LHhiJ15.png)


NO!一个分分钟上万行的文件要人怎么看呢？所以——现在可以切入到另外一种方案：

`service layer`。

Service Layer 基本上是改变了传统 MVC 模式，使用了不太典型的 `n-tiers` 架构：

- 将错综复杂的 domain logic 分拆成一个个的 service，
- 这些 service 之间会共享模型的实现,
- 将传统的 controller 改成 view model，来根据用户的请求组织到 service 的调用。

引入 service layer 另外一个好处就是可以当我们有多个暴露给用户的界面的时候，
可以很方便地保留底层逻辑不变，只需变更 view model 的实现（往往只是更改一下渲染模式）。


而另外一个值得注意的地方就是不同 service 之间应该怎么进行通信？
鉴于 service 是对某个 domain logic 的封装，所以原则上不同 service 之间是不应该有相互**直接**调用的行为的；
而是应该交由上一级的层次来进行有机组织。


#### FYI

`n-tiers` 架构可以由这几部分组成：

- presenter: 针对某些终端的显示方案
- view model: 胶水层，处理数据显示和数据逻辑之间的关系
- service: domain logic 的封装
- model: 实际的 CRUD 操作


----------------------------------

## Recap: Fat Moedel v.s Service Layer

当然咯，具体事情应该具体分析，从上面的描述我们可以看出：

- Fat Model 是对传统 MVC 的一些改进，适合实现交互关系不复杂的业务逻辑。
- Service Layer 是对复杂 domain logic 的封装，适合用于比较大规模的架构。
