---
title: 🧀 mod usage between modules
layout: post
author: cyven
tags: rust
categories: CS CS::Lang CS::Lang::Rust
---




![img](https://s2.loli.net/2024/04/12/h9rKoePILXHdsSJ.png)


Here we create a project named `modtest`

My goal is that functions in `test2` can refer to the functions defined in `test1`

`test1.rs`:
```rust
pub fn test1_foo() {
    println!("test1 foo");
}
```

`test2.rs`:
```rust
mod test1;
use crate::test1::test1_foo;
pub fn test2_foo() {
    test1_foo();
}
```

`main.rs`:

```rust
mod test2;

fn main() {
    test2::test2_foo();
}
```
But it fails in the reason
```rust
-*- mode: compilation; default-directory: "~/Develop/Rust/modtest/" -*-
Compilation started at Fri Apr 12 20:25:06

cargo run --package modtest --bin modtest --
   Compiling modtest v0.1.0 (/Users/mmmmmcclxxvii/Develop/Rust/modtest)
error[E0583]: file not found for module `test1`
 --> src/test2.rs:1:1
  |
1 | mod test1;
  | ^^^^^^^^^^
  |
  = help: to create the module `test1`, create file "src/test2/test1.rs" or "src/test2/test1/mod.rs"
  = note: if there is a `mod test1` elsewhere in the crate already, import it with `use crate::...` instead

error[E0432]: unresolved import `test1::test1_foo`
 --> src/test2.rs:2:5
  |
2 | use test1::test1_foo;
  |     ^^^^^^^^^^^^^^^^ no `test1_foo` in `test2::test1`

Some errors have detailed explanations: E0432, E0583.
For more information about an error, try `rustc --explain E0432`.
error: could not compile `modtest` (bin "modtest") due to 2 previous errors

Compilation exited abnormally with code 101 at Fri Apr 12 20:25:07
```

[defining modules](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#:~:text=Declaring%20submodules%3A)

The doc above tells me that `mod test1` declared in `test2.rs` is nothing more than declaring a **submodule**

So I move `mod test1` to `main.rs`

```rust
mod test1; // added, and removed in test2.rs
mod test2;

fn main() {
    test2::test2_foo();
}
```

It works.

**Submodules must always be declared in the module they reside in !!!**

We have to organize our project into a reasonable hierarchy in such a way that every module have a parent module.

So that we can just declare all the son modules of crate root.
