<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
# 编译器 Atom 安装与适合的插件
>1. [安装 Atom](#安装 Atom "安装 Atom")
1. [常用插件 packages](#常用插件 packages "常用插件 packages")
	1. [1. markdown-preview-plus](#1. markdown-preview-plus "1. markdown-preview-plus")
	1. [2. markdown-writer](#2. markdown-writer "2. markdown-writer")
	1. [3. amWiki](#3. amWiki "3. amWiki")
	1. [4. atom-live-server](#4. atom-live-server "4. atom-live-server")
	1. [5. ralative-numbers](#5. ralative-numbers "5. ralative-numbers")
1. [小结](#小结 "小结")
<!-- /TOC -->

# 编译器 Atom 安装与适合的插件

## 安装 Atom

安装Atom比较简单，从官网上下载最新的版本呢，点击安装就好： [Atom 官网](https://atom.io/)


## 常用插件 packages

因为使用 Atom 主要是为了方便写markdown文档(Atom 默认安装了 markdown 预览(**Ctrl-Shift+M**))和开发个人博客网站 [Glenn's Personal Website](https://glenn-li.github.io)

### 1. [markdown-preview-plus](https://atom.io/packages/markdown-preview-plus)

增强Atom本身自带的预览功能，更加贴近Github的markdown渲染

![效果图](assets/50/600-322598c0.png "效果图")

Adds tons of features to make Atom an even better Markdown/AsciiDoc editor!

### 2. [markdown-writer](https://atom.io/packages/markdown-writer)

![效果图](assets/50/600-7711b0c4.png "效果图")

### 3. [amWiki](https://atom.io/packages/amWiki)

amWiki 是一款基于 Javascript 脚本语言、依赖 Atom 编辑器、使用 Markdown 标记语法的轻量级开源 wiki 文库系统。
amWiki 致力于让大家可以更简单、更便捷的建设个人和团队文库系统！

源码是国产货，又来快速构建个人 wiki 文库系统比较方便，但是有很多用起来不方便的地方，和有一些多有的功能(比如web 服务器)，经过我的裁剪后用起来方便很多了。

我的fork仓库路径 [Glenn's amWiki](https://github.com/Glenn-Li/amWiki)

从我的fork仓库下载代码，然后解压到Atom的packages目录下: C:\Users\yqli\.atom\packages

个人修改的东西：
1、生成markdown文档的目录时，默认插入到页首，如果当前已经存在目录则更新目录，并且把文档的标题(h1)添加到目录上端;
2、首页名字改成 HomePage（可以继续改进成在config.json中指定首页）；
3、把插件的菜单移到Paskages下；
4、取消Web Server功能，目前很不稳定。

![效果图](assets/50/600-a12a3164.png 效果图)

配合Atom+Git的后其中新建和有修改的文件会进行着色处理

![Atom+Git+amWiki](assets/50/600-5b9e2579.png "Atom+Git+amWiki")

### 4. [atom-live-server](https://atom.io/packages/atom-live-server)

用来创建web server的插件，快捷键：

![快捷键](assets/50/600-e66ed2a7.png "快捷键")

个人认为该插件还有待改进，目前打开的web默认是显示网站的入口，但是调试的时候经常是要直接显示到调试页面，所以改进的方向就是设置默认显示页为当前激活的调试页面。

### 5. [ralative-numbers](https://atom.io/packages/relative-numbers)

一个小插件，就是把line number转变成当前的光标所在行的相对位置。

![效果图](assets/50/600-4ca0a8ee.png "效果图")

## 小结

小结是啥，谁给解释一下。
