---
title: Deref和Deref Coercion
layout: post
author: cyven
tags: rust reference String slice deref coercion
categories: CS CS::Lang CS::Lang::Rust
---


## &String==&str???

```rust

fn test(v: &str)->{
	println!("{}",v);
}


fn main(){
	let a = String::from("hello");
	assert_eq!("hello",a);
	test(&a);
}
```

因为在Rust中，我们知道 `hello` 是 str 类型

那这里 `assert_eq!("hello",a)` 会啥可以对比呢？

`test`方法需要 `&str` 但是我们传入 `&String` 为啥又可以？

先说结论,这里涉及到

1. deref
2. deref coercion


## Deref trait

`deref` 是 `Deref` trait 中唯一的方法

### 实现这个trait有什么用呢？？？**

> Implementing the `Deref` trait allows you to customize the behavior of the dereference operator `*`


也就是，比如 `let b=&a`， 在操作 `*b` 的时候，其实会调用定义的`deref()` 方法

```rust

struct MyBox<T>(T);

impl<T> MyBox<T>{
    fn new(v: T) -> Self{
        MyBox(v)
    }
}
use std::ops::Deref;

impl<T> Deref for MyBox<T>{
    type Target = T;

    fn deref(&self) -> &Self::Target{
        &self.0
    }
}

fn main(){

    let a = 5;
    let b = MyBox::new(x);

    assert_eq!(5, a);
    assert_eq!(5, *b);

}

```

### 哪里调用了？

对于任何实现了 `Deref` 特征的类型，compiler在看到 `*b` 的时候，它背后其实是在做 `*(b.deref())`

那 `b.deref()` 返回的是一个引用，`&Self::Target`

那么`*(b.deref())`中的`*`就是正常的解引用

而`deref`方法之所以要返回引用，是因为不让所有权转移



## assert_eq!("hello",a)

`assert_eq!` 是一个宏

```rust
macro_rules! assert_eq {
    ($left:expr, $right:expr $(,)?) => {
        match (&$left, &$right) {
            (left_val, right_val) => {
                if !(*left_val == *right_val) {
                    let kind = $crate::panicking::AssertKind::Eq;
                    // The reborrows below are intentional. Without them, the stack slot for the
                    // borrow is initialized even before the values are compared, leading to a
                    // noticeable slow down.
                    $crate::panicking::assert_failed(kind, &*left_val, &*right_val, $crate::option::Option::None);
                }
            }
        }
    };
	//...
}
```

`match` 里面的通过模式匹配 解构出来的 `left_val` 和`right_val` 两个是 reference ,

因为`match` 的是 `(&xxx,&xxx)`

```rust
	let a = String::from("hello");
	assert_eq!("hello",a);
```

代入`assert_eq!("hello",a)` ,`left_val` 就是 `&str`, `right_val`就是 `&String`

这两个的deref都是定义在std里面

`if !(*left_val == *right_val) {` 这一步就是上面讨论的了

那这里一个是 `&str` 一个是 `&String` ？

因为 String 类型实现的 Deref

`fn deref(&self) -> &str` 它返回的是`&str`


## Deref Coercion

> Coercion: the use of force to persuade someone to do something that they are unwilling to do

这个特性也是Rust编译器会帮助我们实现的，它是在干什么呢？

> Deref coercion converts a reference to a type that implements the Deref trait into a reference to another type.

就好像上面的 `String` 实现了 `Deref` trait，它返回 `&str`

所以

```rust

fn test(v: &str)->{
	println!("{}",v);
}

fn main(){
	let a = String::from("hello");
	test(&a);
}
```

这里当 一个函数的参数需要 `&str` 但我传进来 `&String`的时候，编译器会进行自动转换
注意这里参数类型是 reference

不然的话，我们需要手动  `*` 操作，这里也会涉及到`deref` ，`*(x.deref())`,然后得到值，然后再 `&` 得到值的引用

编译器就帮我们做了这一步了，

>  A sequence of calls to the deref method converts the type we provided into the type the parameter needs.

call deref 这个过程是可以一直下去的，如果需要的话，直到可以找到参数需要的类型


### 重要Demo！！！

```rust
struct MyBox<T>(T);

impl<T> MyBox<T>{
    fn new(v: T) -> Self{
        MyBox(v)
    }
}
use std::ops::Deref;

impl<T> Deref for MyBox<T>{
    type Target = T;

    fn deref(&self) -> &Self::Target{
        &self.0
    }
}

fn hello(name: &str){
	println!("hello,{name}");
}

fn main(){
	let m = MyBox::new(String::from("Rust"));
	hello(&m);
}

```


**这里`hello` 参数需要的是 `&str`**

**但是我们传入了`&MyBox<String>` 类型**

**不过没关系，`MyBox<T>` 实现了 `Deref`**

**所以这里编译器可以帮我们从 `&MyBox<String>` 得到 `&String`,**

**而`&String`与目标`&str` 也不匹配，**

**不过没关系，`String` 实现了`Deref` (std实现的)**

**所以这里编译器可以帮我们从`&String` 得到 `&str`**

从而编译通过


## DerefMut


`Deref` trait 是在 对不可变引用 进行`*` 操作，
如果要对 可变引用进行 `*` 操作，那么需要实现 `DerefMut` trait

那么现在deref coercion可以处理以下三种情况:

- From &T to &U when T: Deref<Target=U>
- From &mut T to &mut U when T: DerefMut<Target=U>
- From &mut T to &U when T: Deref<Target=U>

前两个都很好理解
比较重要的是第三个，但其实这里也就是说明了一点，可以从可变转到不可变，也就是参数申明要的是 `&U`，但我传进来 `&mut T`

但不可以从不可变转到可变。这也很好理解，因为Rust需要保证任何时候最多只有一个可变引用。
