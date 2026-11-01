---
title: 🌳 换零钱算法
layout: post
author: cyven
tags: sicp algorithm
categories: CS CS::Core
---


## 题目

将币值100换成50,25,10,5,1的组合,有几种?


## 分治

这个组合可以分成两个互斥且穷尽的事件组合

- 不使用某一种币值
- 至少使用一次某一种币值

因为不使用和至少使用一次是互斥的

所以我们可以这么拆分

并且解答即两个组合相加

## 一个问题分解为规模更小的同类问题 -- 递归

进而，我们将币值按面值大小排序，然后可以拆分为

- 不使用第一种币值
- 至少使用一次第一种币值

而对于 “不使用第一种币值” 的方案，我们可以看作
“将币值100换成25，10，5，1”,这样的问题

而对于 "至少使用一次第一种币值" ,我们可以看作
"将币值50换成50,25,10,5,1",这样的问题


而上面两个问题是我们一开始的同类问题,使用相同的方式进行拆解

## 停止条件 -- 边界

这是一个递归,那么必须有停机条件

1. 当前要拆换的币值为0,说明我们换完了,且刚好,这时候算1种成功方案
2. 当前要拆换的币值<0,说明当前方案组合出来的币值>100,失败了
3. 当前可以被换的种类没有了,但币值>0, 说明当前方案组合出来的币值<100,失败了


## 代码

```lisp
(define (count-change amount)
  (define (cc amount kind-of-coins)
    (cond ((= amount 0) 1)
          ((< amount 0) 0)
          ((= kind-of-coins 0) 0)
          (else
           (+ (cc (- amount
                     (value kind-of-coins))
                  kind-of-coins)
              (cc amount
                  (- kind-of-coins 1))))))

  (define (value kind-of-coins)
    (cond ((= kind-of-coins 1) 1)
          ((= kind-of-coins 2) 5)
          ((= kind-of-coins 3) 10)
          ((= kind-of-coins 4) 25)
          ((= kind-of-coins 5) 50)))
  (cc amount 5)
  )
```
