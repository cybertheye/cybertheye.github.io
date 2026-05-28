---
title: 🌳 理解Emacs lexical binding 和 dynamic binding
layout: post
author: cyven
tags: emacs
categories: CS CS::Tech
---

## 两个程序例子
### 计数器程序

这里故意用了同名的 test/counter 变量作为全局和局部来说明词法绑定


```lisp
;; -*- lexical-binding: t -*- ;;第一行

(setq test/counter 0)

(defun test/add-2-lambda ()
  (let ((test/counter 0))
    (lambda ()
      (message "inner test/counter is current %d" test/counter)
      (setq test/counter (+ test/counter 2))
      )))

(setq test/add-2-fun (test/add-2-lambda))


(funcall test/add-2-fun)

(funcall test/add-2-fun)

(setq test/counter 0)

(funcall test/add-2-fun)

(funcall test/add-2-fun)

(message "outer test/counter is current %d" test/counter)

```


假如没有第一行的 lexical 绑定, Emacs中默认是动态绑定

那么结果是:

```
inner test/counter is current 0
inner test/counter is current 2
inner test/counter is current 0
inner test/counter is current 2
outer test/counter is current 4

```
可以看到,没有词法绑定的话,  test/add-2-lambda闭包中的 test/counter 变量其实就是访问的外部的 test/counter


有了lexical词法绑定,运行结果是

```
inner test/counter is current 0
inner test/counter is current 2
inner test/counter is current 4
inner test/counter is current 6
outer test/counter is current 0

```

因为开启了 lexical-binding: t，test/add-2-lambda 中的 let 创建了一个严格受限于文本结构的局部词法环境。内部的 lambda 在求值时，将这个局部环境打包成了一个闭包（Closure）。因此，无论你在外部如何使用 (setq test/counter 0) 去修改全局变量，或者无论你在哪里调用 funcall，闭包内部修改的始终是当年被捕获的那个私有的 test/counter，全局的 test/counter 不会受到任何影响


可以改进实验:测试闭包环境的“多实例独立性”


```lisp
(defun make-counter (init-val)
  (let ((counter init-val))
    (lambda ()
      (setq counter (+ counter 2))
      (message "Counter is now %d" counter))))
```


### 定义的位置决定了看到的值

```lisp
;; -*- lexical-binding: t -*-
(setq x 0)

(defun getx ()
  (message "x is %d" x)
  x)

(setq x 3)

(let ((x 2))
  (getx))

```

1. 定义时的级别决定了可见性：你的 getx 函数是在代码的最外层（即顶层、全局级别）定义的。由于它不是在任何 let 等局部绑定结构的内部定义的，因此它的词法环境就是全局环境。它无法“看见”在它外部调用的那些局部变量。

2. 文本范围（Textual Scope）的严格隔离：代码中的 let 表达式创建了一个局部词法环境（绑定了 x = 2）。在词法绑定下，这个 x = 2 仅仅在 let 结构内部的字面文本范围内有效。

3. 运行时的查找机制：当在 let 内部调用 (getx) 时，getx 开始执行。Lisp 解释器在遇到变量 x 时，会首先在 getx 被定义时的词法环境中寻找。因为 getx 是在 let 的外部定义的，它找不到这个局部的词法绑定，于是它就会去全局环境（即符号的 value cell）里获取动态的全局值。

4. 最终的结果：在你调用 let 之前，执行过一句 (setq x 3)，这修改了全局的 x。所以当 getx 去全局环境里找 x 时，它会找到 3。最后这个程序会打印并返回 3。


## 词法绑定到底是什么?

在 Emacs Lisp 传统的动态绑定下，当你使用 let 创建一个局部变量时，这个绑定在 let 表达式执行期间会一直存在于一个全局的“栈”的顶部。这意味着如果你在 let 内部调用了在其他文件中定义的函数，那个外部函数也能看到并修改你的局部变量。 而词法绑定严格限制了变量的作用域：对变量的任何引用都必须位于创建该绑定的代码的文本范围（textual scope）之内

这保证了在 let 内部调用的外部函数绝对无法“看”到你的局部变量
，从而使得程序更加健壮，更不容易出错

## 那动态绑定和java,c/c++中的静态变量有什么区别?

java等语言中的静态变量,修改是破坏性的,不可逆的

Emacs中的动态绑定是维护一个一直存在于一个全局的“栈”的顶部,也就是同名变量如果在其他函数中修改,那么会push一个新的值到这个栈顶,函数结束,那么这个值pop出去,这时候有其他访问的话,访问的旧值


