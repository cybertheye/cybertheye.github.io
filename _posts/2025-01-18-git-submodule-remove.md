---
title: 🧀 如何remove git submodule
layout: post
author: cyven
tags: git submodule
categories: CS CS::Tech
---

git submodule 没有 remove 开箱即用的子命令

所以需要一系列的步骤，这里记录一下(以下是彻底删除)


0. 如果需要备份的话，先备份
1. `git submodule deinit -f path/to/xxx` 这个命令的作用就是会去unregister submodule,`path/to/xxx`下的内容会被情况，然后删除`.git/confg`中关于`path/to/xxx`的信息
2. `git rm -f path/to/xxx` 如何没有加git，需要手动的执行`git add`和`git commit`来记录操作
3. `rm -rf .git/modules/path/to/xxx` 手动的去删除这个目录下的子模块
