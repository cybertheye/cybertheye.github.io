---
title: 🧀 Emacs下取消候选
layout: post
author: cyven
tags: emacs
categories: CS CS::Tech
---


有时候我们会遇到这样的场景,就是在输入状态下,希望将输入的内容作为执行参数

比如创建新文件的时候,我们输入新文件的名字 `init-xxx.el` , 尤其在使用 `C-x C-f` 这个 `find-file` 来创建文件的时候

补全系统这时候可能会匹配到类似的候选项, 尤其在使用 `orderless` 这样的插件的时候, 可能会匹配 `init-xxx-yyy.el`

这时候,按 RET 键会自动选择这个候选项,那么这时候我们其实是希望不采用这个补全功能匹配到的候选项的.

这是因为 `RET`,`C-j`,`C-m` 都是绑定到 `vertico-exit` 这个方法 exit minibuffer with current candidate

有几个技巧

1. `C-p` 或者向上的方向键,将补全高亮回到输入光标处,然后再按回车
2. 如果是 `Vertico` ,可以 `M-RET` , 这个绑定的是 `vertico-exit-input`这个方法表示 exit minibuffer with input,也就是采用input而不是候选. ivy 对应的函数是 `ivy- immediate-done`
