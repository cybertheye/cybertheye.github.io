---
title: 🧩 tustlings tests
layout: post
author: cyven
tags: rust rustlings test
categories: CS CS::Lang CS::Lang::Rust
---



## test5

这里主要搞懂 `unsafe { modify_by_address(&mut t as *mut u32 as usize) };` 就行了
主要是参数

参数中 `*mut u32` 是啥？

`*const T` 和 `*mut T`都是 Rust 中合法的类型，是 raw pointer 类型，原始指针，指向的值的类型是 T

只不过，类似 `let ptr:*const u32 = &a`
在使用 `*ptr`去访问a里面的内容的时候，
因为ptr是原始指针类型，所以这里不能保证地址指向的一定是合法的，所以要用 `unsafe` 包裹

例如

```rust
let my_num: i32 = 10;
let my_num_ptr: *const i32 = &my_num;
let mut my_speed: i32 = 88;
let my_speed_ptr: *mut i32 = &mut my_speed;
unsafe{*my_speed_ptr=77};
println!("my_num:{my_num},my_speed:{my_speed}");
println!("{:?}:{}",my_num_ptr,unsafe{*my_num_ptr});
println!("{:?}:{}",my_speed_ptr,unsafe{*my_speed_ptr});
```

```
my_num:10,my_speed:77
0x7fff3fe67ebc:10
0x7fff3fe67ecc:77
```


然后连续的as就是连续的conversion，

```rust
unsafe fn modify_by_address(address: usize) {
    // TODO: Fill your safety notice of the code block below to match your
    // code's behavior and the contract of this function. You may use the
    // comment of the test below as your format reference.
    unsafe {
        //todo!("Your code goes here")
        let address = address as *mut u32;
        *address = 0xAABBCCDD;
    }
}
```



## test6

这里主要涉及到两个函数，一个是 `Box::into_raw` 一个是 `Box::from_raw`
