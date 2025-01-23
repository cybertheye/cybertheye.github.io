---
title: 🧀 Mac下Emacs手动安装magit
layout: post
author: cyven
tags: mac emacs magit bug
categories: CS CS::Tech
---


参考 [Installing from the Git Repository](https://magit.vc/manual/magit/Installing-from-the-Git-Repository.html)

其他没什么，就是在安装的时候，如果是Mac系统，有可能会出现以下问题

```
┌─(~/.emacs.d/site-lisp/plugins/magit)──────────────────────────────────────────────────────────────────────────────────────────────────────────────────(mmmmmcclxxvii@MMMMMCCLXXVII:s000)─┐
└─(19:52:45 on main)──> make                                                                                                                                                 ──(Tue,Jan14)─┘
Compiling magit-section.el
Compiling magit-base.el
Compiling magit-git.el
Compiling magit-mode.el
Compiling magit-process.e
...
Compiling magit-extras.el
Compiling git-rebase.el
Compiling magit-bookmark.el
 Creating magit-autoloads.el
Checking function declarations
Compiling magit-tests.el
Generating magit.texi
Generating magit-section.texi
Generating magit.info
Generating magit-section.info
Generating dir
Generating magit.html
sed: 1: "/<link rel="stylesheet" ...": command a expects \ followed by text
make[1]: *** [magit.html] Error 1
make: *** [docs] Error 2
```

具体是在处理 HTML 文档中的 CSS 样式链接时 sed 命令执行失败了。*这个问题在 macOS 系统上比较常见，因为 macOS 的 sed 命令与 GNU sed 有些不同。*

解决方案就是安装`GNU sed`

```
#如果你使用 Homebrew
brew install gnu-sed

#然后使用 gsed 替代 sed
PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH" make

```
