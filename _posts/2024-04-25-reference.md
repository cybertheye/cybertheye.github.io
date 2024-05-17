---
title: 🧀 the value of `&&a`
layout: post
author: cyven
tags: rust reference eq
categories: CS CS::Lang CS::Lang::Rust
---


I'm doing the exercise, [rustingls iterators5](https://github.com/rust-lang/rustlings/blob/main/exercises/18_iterators/iterators5.rs)

```rust
fn count_iterator(map: &HashMap<String, Progress>, value: Progress) -> usize {
    // map is a hashmap with String keys and Progress values.
    // map = { "variables1": Complete, "from_str": None, ... }
    //todo!();
    //   &Progress   &&Progress
    map.values().filter(|x| x == &&value).count()
}
```

`values()` returns an iterator of which the element type is `&'a V`.
`filter` will take a `predicate function` as parameter which type is `&Self::item`, that it is also a reference.

So the type of `x` of `|x|` is `&&Progress`

The code passes the tests.

**But my doubt is what's the value of `&&Progress` and what's the difference between `**x==value` and `x==&&value`**

----

I did following test:

[rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=1a4c21e55769abfa3418240fbcf00eb9)

```rust
fn main(){
    let a = 1;
    println!("{:p}",&a);
    println!("{:p}",&&a);
    println!("{:p}",&&a);
}
/*
0x7ffe58e66c0c
0x7ffe58e66ca0
0x7ffe58e66cf0   //not same as previous
*/
```

Printing `&a` is actually getting the address that memory storing the value `1`

But `&&a` ? Since I didn't do such thing like ` b =&a`, that we did allocate a memory space for storing `&a` , it's value seems unpredictable , otherwise it make more sense that getting `&b` is just getting another address like `&a`.

![](https://p.ipic.vip/uydytf.png)


---
Answer from community:

There is no difference between **x == value and x == &&value. The == operator does not compare addresses when you give it a reference. Instead, it just compares the underlying value (but when they meet the same type.)
