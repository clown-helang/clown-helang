---
title: Node：找不到全局安装的模块
catalog: true
date: 2022-04-05 12:43:52
subtitle: 常见问题
header-img: '/img/article_header/article_header.png'
tags: NodeJS
---

## 问题背景

在[Windows 环境下 nvm 的安装与使用](https://www.helang.site/article/node-nvm/)篇章中我们学会了如何使用`nvm`方便快速的切换`node`的版本，但是却发现在写`node`脚本的时候，总是会出现：`Error: Cannot find module 'xxxxxx'`的报错情况。**即使模块已经全局安装过了依旧还是找不到**。这是什么原因又该怎么办呢？

## 问题原因

想要找到原因就需要用到一个非常关键的一个命令：

```shell
# 打印本地目录前缀
npm prefix [-g]
```

> 如果没有`-g`参数，返回的是包含`package.json`文件或`node_modules`目录的最近的**父目录**。如果既没有`package.json`也没有`node_modules`那么就返回**当前文件**所在的目录。
>
> {% asset_img image-20220405113420760.png This is an download page image %}
>
> 如果带有`-g`参数，它就是全局目录前缀。
>
> {% asset_img image-20220405113614478.png This is an download page image %}

对于我们当前遇到的问题就可以使用带`-g`的命令先找到我们的`npm`包全局到底是安装在了哪里。如上图所示，我当前机器上面的全局`npm`包是安装在`C:\Program Files\nodejs`下面。

知道在哪里安装着，下面就看看当前可查找的包都有哪些，可以通过`node`环境下执行`module.paths`查看是否包含该目录：

{% asset_img image-20220405122152302.png This is an download page image %}

从上图可以看出，当前`module.paths`的目录虽然包含有`C:\\Program Files\\nodejs\\lib\\node`，但是不包含`C:\Program Files\nodejs\node_modules`。所以找不到相应的全局包是正常现象。那么，为什么会出现这种情况呢？

我们通过文件管理器打开这个目录看看是什么情况。

{% asset_img image-20220405114303174.png This is an download page image %}

可以看到这里的`nodejs`目录是一个**快捷方式**。打开文件属性查看，其真实目标为`nvm`下面对应的 node 版本的路径。

{% asset_img image-20220405114428408.png This is an download page image %}

至此，我们的问题大致可以确认我们的问题所在：**由于使用`NVM`对`node`进行管理，我们切换了指定的`node`版本导致原来的`NODE_PATH`环境变量发生了变化，所以导致在全局使用的时候找不到相应的`npm`包。**

## 解决办法

既然是因为`NODE_PATH`的问题，那么我们重新配置一下`NODE_PATH`即可。

{% asset_img image-20220405123245627.png This is an download page image %}

配置完成，即可重新打开命令窗口验证问题是否还存在。

{% asset_img image-20220405123813004.png This is an download page image %}

可以看到`module.paths`中已经包含了`C:\Program Files\nodejs\node_modules`目录了。此时再通过`require`引用全局`npm`包也不再报`Error: Cannot find module 'xxxxxx'`的错误了。
