---
title: 🌳 为什么define是special form
layout: post
author: cyven
tags: sicp
categories: CS CS::Core
---


根本原因在于他的求值规则不同于普通函数

普通函数的求值规则是先求出所有子表达式,再将结果应用起来

如果将 define 按照普通函数的求值规则来调用

那么在执行 `(define score 23)` 的时候,会去求 score的值

而在这个阶段, score 没有绑定任何值, 所以会报错

而 define 的作用要求必须拿到 score 这个符号本身,然后对第二个位置进行求值,接着把符号-值这个对应关系保存到环境中. 所以这要求了define必须"绕开现有的求值所有子操作数的这个规则",得采用特定的求值方式.


