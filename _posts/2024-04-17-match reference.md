---
title: Rust match reference
layout: post
author: cyven
tags: rust match ref reference
categories: CS CS::Lang CS::Lang::Rust
---

## 案例

在看[iter_mut](https://rustwiki.org/en/rust-by-example/flow_control/for.html#:~:text=%7D-,iter_mut,-%2D%20This%20mutably%20borrows) 文档的时候

```rust
fn main() {
    let mut names = vec!["Bob", "Frank", "Ferris"];

    for name in names.iter_mut() {
        *name = match name {
            &mut "Ferris" => "There is a rustacean among us!",
            _ => "Hello",
        }
    }

    println!("names: {:?}", names);
}

```

这段代码中的 match 里面的 `&mut "Ferris"` 就挺让我费解的

把 `&mut`去掉就会出现 `mismatched types` 类型错误
把 `"Ferris"` 去掉就会出现 `expected pattern` 错误，因为没提供pattern

因为之前match最多的就是 enum

```rust
enum Color{
	Red,
	Blue,
	//..
}
```

```rust
match color{
	Color::Red => {},
	Color::Blue=> {},
	//...
}
```

好像没有说又要类型又要值的匹配，所以我很困惑

在看了 [pointers/ref](https://rustwiki.org/en/rust-by-example/flow_control/match/destructuring/destructure_pointers.html#:~:text=%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-,pointers/ref,-For%20pointers%2C%20a) 之后懂了

> For pointers, a distinction needs to be made between **destructuring and dereferencing** as they are different concepts which are used differently from languages like C/C++.

其实就是你 match 后面如果是个引用，但你不加 `*`的话 ，匹配的时候就要加上 `&`

它这里也给了例子

```rust
	let reference = &4;
    match reference {
        // If `reference` is pattern matched against `&val`, it results
        // in a comparison like:
        // `&i32`
        // `&val`
        // ^ We see that if the matching `&`s are dropped, then the `i32`
        // should be assigned to `val`.
        &val => println!("Got a value via destructuring: {:?}", val),
    }
```

`reference` 是 `&i32`类型的，去匹配`&val`，就好像消消乐一样，大家都先把`&`消了，

然后把 `i32`类型的值给`val`，**这是 destructure 的概念**

这是一种方式，还有另一种方式,就是match的时候带上 `*`

```rust
//直接用前面的例子

fn main() {
    let mut names = vec!["Bob", "Frank", "Ferris"];

    for name in names.iter_mut() {
        *name = match *name {
            "Ferris" => "There is a rustacean among us!",
            _ => "Hello",
        }
    }
    println!("names: {:?}", names);
}

```

可以这么写， 现在是`match *name`了，然后就可以把模式匹配部分的`&mut`给去掉了

**只不过这里的模式匹配其实不是 destructure,因为没有涉及到用一个变量去接受值**


## Key Point

区分 什么是 destructure 和 dereference
