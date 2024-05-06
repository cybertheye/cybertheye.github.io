---
title: Rust多线程中的生命周期
layout: post
author: cyven
tags: rust concurrency thread lifetime reference
categories: CS CS::Lang CS::Lang::Rust
---

```rust
use std::thread;
fn main(){
	let v = vec![1,2,3];

	let handle = thread::spawn(||{
		println!("here's a vector:{:?}",v);
	});

	handle.join().unwrap();
}

```

这里因为 `println!()`函数只需要 v 的引用，如果只是单线程的话，完全没有任何问题。

而一看到引用，就要想到谁是 本体， 本体获得够不够久

而现在因为这段代码跑在不同的线程上，这就导致了一个问题，就是引用本体，v会不会提前嗝屁了。

虽然这里我们直观看，是没有的，因为用了 `join()` 去等待，

但是Rust并不知道，或者说 不好推断

如果是这样

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}

```

那就会出问题了


