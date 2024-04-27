---
title: rustlings - iterators
layout: post
author: cyven
tags: rust
categories: CS CS::Lang CS::Lang::Rust
---



## iterators1



## iterators2

```rust
// Step 1.
// Complete the `capitalize_first` function.
// "hello" -> "Hello"
pub fn capitalize_first(input: &str) -> String {
    let mut c = input.chars();
    match c.next() {
        None => String::new(),
        Some(first) => first.to_uppercase().to_string() + c.as_str(),
    }
}

// Step 2.
// Apply the `capitalize_first` function to a slice of string slices.
// Return a vector of strings.
// ["hello", "world"] -> ["Hello", "World"]
pub fn capitalize_words_vector(words: &[&str]) -> Vec<String> {
    let mut vecr = Vec::new();
    for word in words {
        vecr.push(capitalize_first(word));
    }
    vecr
}

// Step 3.
// Apply the `capitalize_first` function again to a slice of string slices.
// Return a single string.
// ["hello", " ", "world"] -> "Hello World"
pub fn capitalize_words_string(words: &[&str]) -> String {
    let mut ret = String::new();
    for word in words {
        ret = ret + capitalize_first(word).as_str();
    }
    ret
}
```

我本来是这么写的。然后在看了hint之后，提到了collect，然后去阅读了一下文档

那些能通过一个 `iterator` 得到其他 `iterator` 的方法叫做 `iterator adaptors`，例如 `map`
那些消费 `iterator` 的方法叫做 `consuming adaptors` ,例如  `sum`, `collect`

collect 的作用就是 **collects the resulting values into a collection data type**

所以`capitalize_words_vector`方法可以这样写

```rust
pub fn capitalize_words_vector(words: &[&str]) -> Vec<String> {
	words.iter().map(|x| capitalize_first(x)).collect()
}
```

那么step3 中其实可以利用 step2，Vector中有一个  `join` 方法

```rust
pub fn capitalize_words_string(words: &[&str]) -> String {
    capitalize_words_vector(words).join("")
}
```



## iterators3

### divide 方法比较简单

```rust
pub fn divide(a: i32, b: i32) -> Result<i32, DivisionError> {
    //todo!();
    if b == 0 {
        Err(DivisionError::DivideByZero)
    } else if a % b == 0 {
        Ok(a / b)
    } else {
        Err(DivisionError::NotDivisible(NotDivisibleError {
            dividend: a,
            divisor: b,
        }))
    }
}
```

### `result_with_list`

原来我只是轻蔑的一笑，噗，就这
然后我指尖开始舞动
```rust
fn result_with_list() -> Result<Vec<i32>, DivisionError> {
    let numbers = vec![27, 297, 38502, 81];
    let division_results = numbers.into_iter().map(|n| divide(n, 27))?;
	Ok(division_results.collect())
}
```

当看到编译器报错的时候，我意识到事情没有那么简单

我原以为，这里就用 error propagation 就可以了，但是错误说
```rust
47 |     let division_results = numbers.into_iter().map(|n| divide(n, 27))?;
   |                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the `?` operator cannot be applied to type `Map<std::vec::IntoIter<i32>, {closure@exercises/iterators/iterators3.rs:47:52: 47:55}>`
   |
   = help: the trait `Try` is not implemented for `Map<std::vec::IntoIter<i32>, {closure@exercises/iterators/iterators3.rs:47:52: 47:55}>`
```
然后老老实实写
```rust
fn result_with_list() -> Result<Vec<i32>, DivisionError> {
    let numbers = vec![27, 297, 38502, 81];
    let division_results = numbers.into_iter().map(|n| divide(n, 27));
    let mut ret = Vec::new();
    for result in division_results {
        match result {
            Ok(x) => ret.push(x),
            Err(e) => return Err(e),
        }
    }
    Ok(ret)
}
```

这样肯定对，但总觉得应该有更好的写法

然后读了 [collect](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect) 这个文档后

> collect() can also create instances of types that are not typical collections. For example, a String can be built from chars, and an iterator of Result<T, E> items can be collected into Result<Collection<T>, E>.

```rust
let results = [Ok(1), Err("nope"), Ok(3), Err("bad")];

let result: Result<Vec<_>, &str> = results.iter().cloned().collect();
```

1. list中如果有 Err的话，result是第一个error，也就是 `Err("nope")`

```rust
let results = [Ok(1), Ok(3)];

let result: Result<Vec<_>, &str> = results.iter().cloned().collect();
```

2. 都是`Ok(T)`的话，result是 `Ok(Vec<T>)`,也就是 `Ok([1,3])`

因为其实这里申明了 type， `result: Result<Vec<_>,&str>`
除了error的处理会有点特殊

那么上面其实就可以写成
```rust
fn result_with_list() -> Result<Vec<i32>, DivisionError> {
    let numbers = vec![27, 297, 38502, 81];
    let division_results = numbers
        .into_iter()
        .map(|n| divide(n, 27))
        .collect::<Result<Vec<i32>, DivisionError>>();
    division_results
}
```

那可不可以用 `?` 呢？
```rust
fn result_with_list() -> Result<Vec<i32>, DivisionError> {
    let numbers = vec![27, 297, 38502, 81];
    let division_results = numbers
        .into_iter()
        .map(|n| divide(n, 27))
        .collect::<Result<Vec<i32>, DivisionError>>()?;
    Ok(division_results)
}
```

### `list_of_results`
```rust
fn list_of_results() -> Vec<Result<i32, DivisionError>> {
    let numbers = vec![27, 297, 38502, 81];
    let division_results = numbers.into_iter().map(|n| divide(n, 27)).collect();
    division_results
}
```

和上面比 只是 返回的 type 不一样， collect 会做剩下的事


## iterators4

```rust
pub fn factorial(num: u64) -> u64 {
    (1..=num).product()
}

```

这里 `(a..=b)`  如果 `b<a`, 那么其实空范围，此时 `product` 方法返回1

因为`product` 就是计算相乘，肯定从1开始，难不成从0开始，那不就都是0了

`1*a*(a+1)...*b`


hint中还提到了`fold` 函数

```rust
pub fn factorial(num: u64) -> u64 {
    (1..=num).fold(1, |acc, x| acc * x)
}

```

 `fold`函数
```rust
 fn fold<B, F>(self, init: B, f: F) -> B
where
    Self: Sized,
    F: FnMut(B, Self::Item) -> B
```

 其实给了更自由的，因为可以传入一个闭包函数，可以自定义对 `accumulator`的操作

 闭包包含两个参数，一个是`acc`,一个是`x` 前者就是accumulator，累积操作的值，它的初始值是`init`参数指定的


`reduce`方法其实就是初始值是 first element 不需要指定
