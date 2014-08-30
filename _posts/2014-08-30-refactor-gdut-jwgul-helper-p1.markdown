---
layout: post
title: "手把手教你重构教务助手 （一）"
date: 2014-08-30 13:13:00
author: hbc
categories: programming javascript
---

## 目录

- [第一天：准备开发环境]()
- [第二天：第一个功能！和第一次重构]()
- [第三天：成绩查询与创造环境]()


-------------------------------------

## 起因


[广工大教务管理助手](http://projects.vtmer.com/gdut-jwgl-helper/) 是 [@Link](https://github.com/linkkingjay) 酱在发现鄙校的[教务系统](http://jwgl.gdut.edu.cn) 超级难用之后怒写的浏览器插件。


因为最开始的时候需要解决的问题比较简单，所以写出来的效果就是 "it just works"。
但这么好用的东西肯定很抢手啊！前些天插件也收到了第一个来自工作室外部的 [pull request](https://github.com/vtmer/gdut-jwgl-helper/pull/17)。


但有点不太妙的就是这个 pr 在添加了一系列很赞的功能的同时，添加了不少古怪的 bug -\_- 。
归根到底还是原来插件的设计架构没有很好地传达给其他的代码阅读者（这里我是脑补的）；
所以重构一下就显得非常迫切了。为此接下来几篇文章将会记录下脚本的重构过程，方便后续的开发……


----------------------------------


## 开发环境准备


### 目录准备

为了直播本次重构过程，我重新组织了一下项目的目录结构如下：

```
.
├── ext
│   ├── gdut-jwgl-helper.js
│   ├── jquery-1.8.3.min.js
│   └── manifest.json
├── gdut-jwgl-helper.crx
├── gdut-jwgl-helper.js
├── gdut-jwgl-helper.pem
├── gm2chrome
│   ├── converter.py
│   ├── grantGM_xmlhttpRequest.js
│   └── README.mkd
├── image
│   ├── check.png
│   ├── GPA.png
│   └── rank.png
├── Makefile
├── README.md
└── refactor  <- 我们在这里面开干
```


### 自动化脚本


古话有云，磨刀不误砍柴工。所以在开始干活之前，我们要准备一下开发环境。

第一版的助手使用了 [Makefile](https://github.com/vtmer/gdut-jwgl-helper/blob/cbc7f365437156a9c5fb26c1648a28c3c17794e0/Makefile) 和一个[油猴转 chrome 插件脚本](https://github.com/bcho/gm2chrome) 来组织开发。

它们提供的功能是：

- 自动转换、打包插件的源码脚本
- 自动生成油猴、chrome 浏览器两种不同格式的插件


虽然这套方案能用，但还需要假定你在 Linux 环境上操作，并且配置一下 python 环境。（谁叫我还没钱买水果电脑呢 :-|）
因而会让其他人觉得超（一）级（头）科（雾）幻（水）。


所以这次重构首先做的就是把这套 homebrew 的方案替换成更为前端开发者熟悉的 `grunt` 工具。

----------------------------------

##### FYI: Grunt 是什么鬼东西？

[Grunt](http://gruntjs.com/) 是一个由一些很厉害的前（金）端（轮）大（法）师（王）开发的自动化任务管理工具（`task runner`），
旨在帮助不会写 Makefile 的各位前端写出很有 javascript 味道的自动化管理脚本。（好吧后半句又是我脑补的）


Grunt 本身只提供简单的文件管理、任务定义功能；而通过组合其他插件，可以实现很多方便的任务。（例如前端文件流水线处理、livereload）


同这个东西类似的还有 [Gulp](http://gulpjs.com/) 。（果真是地球上轮子最多的社区 O.O）

----------------------------------

#### 初始化一个 npm 包开发环境

为了让其他人重现我们的开发环境，我们需要预先初始化一个 npm 包开发环境：

```bash
$ npm init

This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sane defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
name: (refactor) gdut-jwgl-helper
version: (1.0.0) 0.2.0
description: Make jwgl.gdut.edu.cn better.
entry point: (index.js) 
test command: 
git repository: 
keywords: chrome extension, greasemonkey script
author: Link <linkkingjay@gmail.com>, hbc <bcho@vtmer.com>
license: (ISC) MIT
About to write to /some/where/gdut-jwgl-helper/refactor/package.json:
```

运行后即可以看到一个 [package.json](https://github.com/vtmer/gdut-jwgl-helper/blob/4c17c560124cacc56eef6d973d4ebdd2b84f6769/refactor/package.json) 文件，里面包含了助手开发需要的依赖。其他人重新准备环境的时候只需要执行：

```bash
$ npm install
```

即可。


#### 安装 Grunt 以及它的小伙伴

```bash
$ npm install grunt --save-dev
$ npm install grunt-contrib-{watch,clean,copy,concat} grunt-shell --save-dev
```

**注意**: `--save-dev` 这个选项记得加噢，不然就不能把这些依赖写入到 `package.json` 里面去了。

----------------------------------

##### FYI: package.json 又是什么鬼？

看[这里](https://www.npmjs.org/doc/files/package.json.html)。

----------------------------------

#### 写任务脚本！

搞了半天，我们终于要开始写第一行代码了。在开始之前，我们先规划一下整个项目的目录环境：

```
.
├── build                           <- 打包出来的临时目录
│   ├── crx                           <- chrome 插件打包的临时目录
│   ├── crx.crx                       <- 打包出来的 chrome 插件
│   └── gdut-jwgl-helper.gm.js        <- 打包出来的油猴脚本
├── docs                            <- docco 生成的文档
├── gdut-jwgl-helper.0.2.0.crx      <- 生成的正式 chrome 插件
├── gdut-jwgl-helper.0.2.0.js       <- 生成的正式油猴脚本
├── Gruntfile.js                    <- Grunt 配置脚本
├── package.json                    <- 项目定义
└── src                             <- 源码 ( ^ิ౪^ิ)
    ├── gdut-jwgl-helper.js           <- 真正的脚本
    ├── manifest                      <- 各种配置文件
    │   ├── crx.json                    <- chrome 插件打包的配置
    │   ├── gdut-jwgl-helper.pem        <- chrome 插件打包的密钥
    │   └── greasemonkey.js             <- 油猴脚本打包的配置
    └── vendor                        <- 第三方脚本（jquery 能治百病）
        └── jquery.min.js
```

整个自动脚本的执行过程就是：

1. 开发时检查我 src 里面文件的变化
  - 发生变化？打包脚本到 build 目录中
    * 油猴脚本：将油猴脚本的配置和脚本源码拼接起来，输出到 build 目录
    * chrome 插件：将 chrome 插件打包配置和脚本源码复制到 build/crx 目录

2. 打包脚本并发布
  - 油猴脚本：将拼接好的脚本复制到当前目录，并重命名为 gdut-jwgl-helper.<version>.js
  - chrome 插件：
    * 调用 chrome 浏览器，使用命令行接口把 build/crx 内的文件打包成一个 crx 插件
    * 将打包好的插件复制到当前目录下，并重命名为 gdut-jwgl-helper.<version>.crx


具体的脚本可以参考 [Gruntfile.js](https://github.com/vtmer/gdut-jwgl-helper/blob/4c17c560124cacc56eef6d973d4ebdd2b84f6769/refactor/Gruntfile.js)。

写好脚本之后我们就可以使用如下的工作流啦：

* 开发：
  - grunt
  - 改改改
  - 到浏览器里刷新看效果

* 部署：
  - grunt publish
  - git add & commit


到这里，我们已经基本准备好开发环境了；接下来就可以开心地写代码啦。
