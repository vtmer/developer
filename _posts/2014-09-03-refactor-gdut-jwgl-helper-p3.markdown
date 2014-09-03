---
layout: post
title: "手把手教你重构教务助手 （三）"
date: 2014-09-03 21:42:00
author: hbc
categories: programming javascript
---

## 目录

- [第一天：准备开发环境](http://developer.vtmer.com/2014/08/refactor-gdut-jwgl-helper-p1.html)
- [第二天：第一个功能！和第一次重构](http://developer.vtmer.com/2014/09/refactor-gdut-jwgl-helper-p2.html)
- [第三天：课程评价与创造环境](http://developer.vtmer.com/2014/09/refactor-gdut-jwgl-helper-p3.html)


## Repeat Yourself

虽然计算机圈里面有句很经典的名言叫：[Don't repeat yourself, DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)，
但请不要把这个乱用噢。很多时候为了做到代码上的 DRY，你还是需要 repeat yourself 来做一些固定的事情的。

测试驱动开发就是一个很好的例子。在（二）中我们已经完整地搭建起一个测试环境，和走了一轮测试驱动的流程，
接下来就可以 repeat 这个 flow 开发新的功能：课程评价。

噢你说你已经忘记了这个 flow 是什么？好跟我读一遍（应该罚你读 100 遍）：

- 在写代码之前先思考
- 写测试
- 写代码来让你的测试通过
- 认真写你的代码


（十分钟过去……）

经过十分钟的思考，我们可以确定课程评价需要这样的功能：

> 帮我一！键！评！价！


So 接下来我们可以开始考虑怎么来一键评价了。经过细心研究页面结构，发现那一堆的选项其实就是 `select` （这不是废话么）。
所以我们要做的就是给每个 select 都在**指定数据范围**生成一个选项 —— 就是一个随机数生成的问题咯，那么下面请拿出你的笔和纸来推公式！


（五分钟过去……（没错我数学差））

我得出了如下的公式：

    选择的值 = Math.floor(Math.random() * 有多少个选项可以选？) + 起始值


另外需要注意的就是要保证不是所有的序列选项都一样噢。在有了上述的认识之后，我们可以写出如下的测试（也可以叫 sepc 啦）：


[代码](https://github.com/vtmer/gdut-jwgl-helper/blob/475fbb69f680b03a48f765836f2e9e27e545059a/refactor/src/test/rating_maker.js)


接下来就是要完成 ``RatingMaker`` 这个家伙了：

[代码](https://github.com/vtmer/gdut-jwgl-helper/blob/475fbb69f680b03a48f765836f2e9e27e545059a/refactor/src/gdut-jwgl-helper.js#L251,L271)


还有……加载到页面中：

[代码](https://github.com/vtmer/gdut-jwgl-helper/blob/475fbb69f680b03a48f765836f2e9e27e545059a/refactor/src/gdut-jwgl-helper.js#L379,L436)


接着打开我们的 mocha 测试页……等待 1 秒钟——

![It Works!](http://i.imgur.com/2yzWuGN.jpg)


## 没有条件也要创造条件上

但是你会说……这功能得在教务系统里进行测试吧？但我又已经把所有课程都一个一个评价点完（是不是很恨你的老师），
不能进入这个评价入口了噢。


这种情况下，我们就可以创造（mock）出一个比较简单但基本相同的测试环境来检查啦：

[测试页面代码](https://github.com/bcho/gdut-jwgl-helper/blob/08894c9ec4f0b078f8d20294d33aba0a46a44f84/refactor/src/test/rating.html)

--------------------------

##### FYI 什么鬼是 mock？

请参考[这里](http://en.wikipedia.org/wiki/Mock_object)。

--------------------------

好，让我们屏息凝气，认真点每个按钮……

![woah](https://camo.githubusercontent.com/92a0e6c8cfa3aae0e70f442fc915263ee59bf5bf/687474703a2f2f6d656469612e74756d626c722e636f6d2f74756d626c725f6c74757a6a766251364c31717a677078392e676966)

![test-failed](/images/refactor-gdut-jwgl-helper-p3/test-failed.png)


难道我的数学老师真正身份是体育老师？！


但明明针对 `RatingMaker` 的测试是都通过的了呀……于是我们可以将矛头转到具体的页面操作代码上。经过地毯式的排查，我们终于发现原来是踩中了每个 pro(stupid) programmer 都会遇到的地雷： off-by-one 。


修复之后，我们可以再次查看效果：

![test-success](/images/refactor-gdut-jwgl-helper-p3/test-success.png)


赞！


--------------------------

##### 问： 什么鬼是 off-by-1 bug？

答：There are 2 hard problems in computer science: caching, naming, and off-by-1 errors.

--------------------------


## Recap

到这里，我们已经波澜不惊地完成了广工大教务管理助手的[重构](https://github.com/vtmer/gdut-jwgl-helper/pull/19)。
在这个过程中，我们用到的方法和工具有：

- grunt -> 自动构建、测试工具
- mocha, should.js -> BDD 风格的测试工具
- javascript -> 代码
- TDD -> 减少刷新浏览器
- 实现和调用分离 -> 减少代码耦合，方便进行测试
- mock -> 方便处理代价高昂的情况
- docco -> 文档生成


看到这里，是不是感觉要跃跃欲试开始添加新功能了呢？[Welcome](https://github.com/vtmer/gdut-jwgl-helper/issues/new)！
