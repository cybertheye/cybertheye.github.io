---
title: 🌳 treesit-auto 介绍
layout: post
author: cyven
tags: emacs treesit
categories: CS CS::Tech
---


## 什么是treesit

treesit 是一种语义分析工具

比如代码的高亮,怎么决定哪些是要高亮的呢?

比如代码的折叠,怎么决定哪到哪是可以折叠的呢?

比如代码的括号的跳转,怎么知道哪个和哪个括号是匹配的可以跳转的呢?

这背后就需要 treesit 的支持

## 为什么要用treesit

那么 Emacs 在没有支持 treesit 的时候, 普通的 mode 打开, 比如 python-mode 打开 python 文件,以上都是利用正则表达式来完成工作.

利用正则表达式,首先是性能上会差,其次是扩展性不好.因为它本质是根据字符串匹配,而不是语义分析来的.


## 如何配置 treesit

Emacs29 之后就内置了 treesit, 也有一些 mode 已经是 xxx-ts-mode 了, 加了 -ts- 的说明当前这个 mode 是利用treesit来做语义解析的

1. 首先是安装 language-grammar, 会去调用这些程序做语义解析,相当于 server 端
2. 我们给代码文件关联 major mode , 比如 python 文件,我们需要用 python-mode 或者 python-ts-mode 来打开吧,需要关联起来,因为 Emacs 内置了一些 mode,有些还是 ts-mode, 但是没有的话,需要去 github 上找一下,然后下载这些 plugin . 具体可以直接添加到 `auto-mode-alist`中,比如`(push '("\\.java$" . java-ts-mode) auto-mode-alist)` 这样就会在打开`.java`文件的时候自动开启 `java-ts-mode`. 也可以对一些已经有的 mode 做一些 remap , 映射到 ts-mode, 比如 `(push '(rust-mode . rust-ts-mode) major-mode-remap-alist)`
3. 虽然一些Emacs 内置了不少解析器， 很多语言默认就打开了 Tree-Sitter 的支持，但是一些新加的语言 Emacs 还没有配置，这时候需要使用 treesit-parser-create 来手动开启 Tree-Sitter, `(add-hook 'markdown-ts-mode-hook #'(lambda () (treesit-parser-create 'markdown)))`

## treesit-auto又是什么?

很简单, [treesit-auto](https://github.com/renzmann/treesit-auto) 就是帮你做了上面的事情的一个包
