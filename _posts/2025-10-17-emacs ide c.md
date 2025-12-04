---
title: 🧀 Emacs代码编写工作流
layout: post
author: cyven
tags: ide emacs
categories: CS CS::Lang
---

# 理一下写代码的环境的需求

其实写代码，最早也就是在文本编辑器里面就可以了，只是现代IDE封装了很多便携的操作

因为我们是Emacser，我们要自己构建这样的IDE，这样才酷

那么有文本编辑功能就可以写代码，但是在写代码的时候可能需要一些辅助功能，就比如，智能补全,代码高亮，错误提示，以及跳转
这几个是我认为的最基本的

然后就是，编译代码，比如C/C++的程序在命令行使用CMake就行，但在Emacs中有没有更高效的使用姿势？
不能每次都转到命令行，然后手动，这太不优雅了

编译完就是执行了，如何能一键执行

这就是基本的需求了


# 构建环境

## mode

这个写哪个语言就需要先有对应语言的 major mode

## 智能补全,代码高亮,错误提示

这两个都是 LSP 的火力覆盖范围,我是用的猫大的 lsp-bridge
因为 lsp-bridge 有自带的 lsp-bridge-diagnostic-xxx , 所以这里我没有使用 flychecker 等其他错误诊断框架,
不然还要安装 checker 就很多余.

不过 yasnippet 还是有必要安装一下的

## 跳转

其实有多种方式可以实现跳转

### 1.LSP

我使用的是猫大的 `lsp-bridge`, 就比如 `lsp-bridge-find-reference` ,让光标停在一个变量或者函数名上,就可以跳转到定义的地方

### 2.项目管理

本质上就是搜索,只是在项目文件内搜索, 比如 `projectile-riggrep`

但是这种方式是缺少的代码语义

### 3.第三方工具

比如 c/c++ 使用 `ctags`或者`cscope`


## 编译,执行

其实本质是命令行调用,在 Emacs 中可以结合 projectile

因为 projectile 提供了`projectile-configure-project`, `projectile-compile-project`, `projectile-run-project`等方法

## 重构

使用猫大的 color-rg ,项目中搜索,然后替换. That's it

