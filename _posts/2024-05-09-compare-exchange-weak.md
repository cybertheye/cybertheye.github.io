---
title: 🧩 Rust - atomic中的compare_exchange_weak
layout: post
author: cyven
tags: rust atomic platform x86 arm cas ll sc
categories: CS CS::Lang CS::Lang::Rust
---

在看Rust有关atomic的文档的时候，有一个 `compare_exchange_weak` 方法

```rust
pub fn compare_exchange_weak(
    &self,
    current: bool,
    new: bool,
    success: Ordering,
    failure: Ordering
) -> Result<bool, bool>
```

它和 `compare_exchange` 方法

```rust
pub fn compare_exchange(
    &self,
    current: bool,
    new: bool,
    success: Ordering,
    failure: Ordering
) -> Result<bool, bool>
```

函数签名完全一样，那么他俩的区别是什么呢？

## 首先了解一下compare_exchange是做什么的？

他做的事情就是 compare and swap ， 看一下current value 是不是预期的，如果是那么改成 new value，不然的话就失败，而这些是一个原子操作

为什么是一个原子操作呢？是因为底层平台的支持

它和 `compare_exchange_weak` 的区别就是，后者除了 current value不是预期 导致的失败，后者还允许其他原因导致的失败，

还有能有其他原因？？？

## 了解一下LL/SC

比如 x86中的 cas 指令 CMPXCHG，

但是比如在ARM架构中，他没有直接像 x86 这样的 cas指令， 取而代之的是 [load-link/store-conditional](https://en.wikipedia.org/wiki/Load-link/store-conditional) **两对指令**

load-link 你可以想成就是获取这个内存空间的独有的所有权，然后返回这个内存空间的值

**store-conditional 会写入一个新值，但是！！！ only if, 只有在 ，load-link和这个store-conditional操作之间，这个内存空间没有更新过**

**有一种情况是，其他线程去更新这个内存空间，但是是同样的值，这样store-conditional 也会失败**

**这也是ABA问题，而这种问题是 x86 架构 cas 操作无法察觉的**

甚至 another load-link也会导致store-conditional失败

> Real implementations of LL/SC do not always succeed even if there are no concurrent updates to the memory location in question. Any exceptional events between the two operations, such as a context switch, another load-link, or even (on many platforms) another load or store operation, will cause the store-conditional to spuriously fail.

所以ARM架构是依靠 循环 ll/sc 这样组合来完成 cas 做的事情

## 失败的情况

所以在 `compare_exchange_weak` 文档里面可以看到这句话

> Unlike AtomicBool::compare_exchange, this function is allowed to spuriously fail even when the comparison succeeds, which can result in more efficient code on some platforms.

所以这里的spuriously fail就是指的上面说的那些情况

因为compare_exchange 只有在 current value 不是 预期的时候失败

而在不同平台下，就比如 ARM ，导致它失败的原因 可能有其他，上面说过了，（我突然想到了数学中，过程不对，结果正确的情况）

而`compare_exchange_weak` 就能允许捕捉到这种失败而提前返回失败，那这样控制权就又交回了我们自己代码手中，从而可以执行自己的逻辑

在x86下 `compare_exchange_weak` 还是cas指令去完成的

## 循环重试

那如果用 `compare_exchange` 呢，因为说了在例如 ARM 架构中是通过 不断的  循环 ll/sc  两对指令来实现cas操作的，那么如果不像  `compare_exchange_weak`那样允许捕捉到 除了 current value不是预期值的失败，那么它就只能不断的循环重试， 这样就是所谓的效率不高了么

而且，如果我们的代码是这么写的话

```rust
while lock
	  .compare_exchange(UNLOCKED,LOCKED,Ordering::Relaxed,Ordering::Relaxed)
	  .is_err()
{
	//
}
```

那么在ARM下就是循环套循环


## 正确的姿势

那么如果你已经用了一个循环，那么最好选择 `compare_exchange_weak`

那么如果没有用循环，

   如果你只在乎结果，那么 `compare_exchange` 就可以

   如果你也在乎过程正义，那么选择 `compare_exchange_weak`， 毕竟 `is_err()` 可以提前返回 false了
