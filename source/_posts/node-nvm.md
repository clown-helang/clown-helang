---
title: Windows环境下nvm的安装与使用
catalog: true
date: 2022-03-14 23:05:09
subtitle: 日常工具
header-img: '/img/article_header/article_header.png'
tags: NodeJS
---

## 简介

`nvm`允许你通过命令快速安装和使用不同版本的`node`。

**例如**

```sh
$ nvm use 16.9.1
Now using node v16.9.1 (npm v7.21.1)
$ node -v
v16.9.1
$ nvm use 14.18.0
Now using node v14.18.0 (npm v6.14.15)
$ node -v
v14.18.0
$ nvm install 12.22.6
Now using node v12.22.6 (npm v6.14.5)
$ node -v
v12.22.6
```

就是这么简单。

## 关于

`nvm` 是[node.js](https://nodejs.org/en/)的版本管理器，其主要功能是可以在本地安装多个版本的 nodejs，且可切换版本。如果读者使用的是 MacOS 或 Linux 可以参考 [nvm-sh/_nvm_](https://github.com/nvm-sh/nvm) 项目，如果是 windows 的话则参考 [coreybutler/_nvm_-windows](https://github.com/coreybutler/nvm-windows) 项目。由于笔者本人使用 windows，所以这里下载安装以后者为例。

## 下载与安装

下载地址：[V1.1.9](https://github.com/coreybutler/nvm-windows/releases/tag/1.1.9)

{% asset_img image-20220314224225497.png This is an download page image %}

下载后解压并执行`nvm-setup.exe`

{% asset_img image-20220314224514959.png This is an install image %}

{% asset_img image-20220314224551906.png This is an install image %}

一路`next`即可。

## 环境变量配置

正常情况下通过`nvm-setup.exe`安装成功后即可使用，如果出现`nvm`命令找不到情况，需要配置环境变量。可参考下图配置：

{% asset_img image-20220314225257680.png This is an image %}

{% asset_img image-20220314225047676.png This is an image %}

{% asset_img image-20220314225149693.png This is an image %}

## NVM 源配置

找到`nvm`安装目录，打开`settings.txt`文件，如果没有可以自己创建一个。

{% asset_img image-20220314225733651.png This is an image %}

然后打开并编辑：

```bash
root: D:\FE\nvm
path: C:\Program Files\nodejs
arch: 64
proxy: none
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```

## 验证

打开终端工具，输入：`nvm`

{% asset_img image-20220314230129171.png This is an image %}

如果出现如上图所示内容，那么恭喜你安装成功了。

## 常用命令

```bash
# 列出所有可以安装的node版本号
nvm list available
# 安装指定版本号的node
nvm install xxx
# 列出所有已经安装的node版本
nvm list/ls
# 切换node的版本
nvm use	xxx
```

## 参考文章

- https://www.cnblogs.com/oneclue/p/14915977.html
- https://github.com/nvm-sh/nvm/blob/master/README.md
