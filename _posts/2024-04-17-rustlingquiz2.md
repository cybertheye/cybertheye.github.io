---
title: 🧩 rustlings quiz
layout: post
author: cyven
tags: rust
categories: CS CS::Lang CS::Lang::Rust
---

## quiz2 题目
rustlings的题目好像会变。现在做的这个quiz2的题目是

```rust
// quiz2.rs
//
// This is a quiz for the following sections:
// - Strings
// - Vecs
// - Move semantics
// - Modules
// - Enums
//
// Let's build a little machine in the form of a function. As input, we're going
// to give a list of strings and commands. These commands determine what action
// is going to be applied to the string. It can either be:
// - Uppercase the string
// - Trim the string
// - Append "bar" to the string a specified amount of times
// The exact form of this will be:
// - The input is going to be a Vector of a 2-length tuple,
//   the first element is the string, the second one is the command.
// - The output element is going to be a Vector of strings.
//
// No hints this time!

// I AM NOT DONE

pub enum Command {
    Uppercase,
    Trim,
    Append(usize),
}

mod my_module {
    use super::Command;

    // TODO: Complete the function signature!
    pub fn transformer(input: ???) -> ??? {
        // TODO: Complete the output declaration!
        let mut output: ??? = vec![];
        for (string, command) in input.iter() {
            // TODO: Complete the function body. You can do it!
        }
        output
    }
}

#[cfg(test)]
mod tests {
    // TODO: What do we need to import to have `transformer` in scope?
    use ???;
    use super::Command;

    #[test]
    fn it_works() {
        let output = transformer(vec![
            ("hello".into(), Command::Uppercase),
            (" all roads lead to rome! ".into(), Command::Trim),
            ("foo".into(), Command::Append(1)),
            ("bar".into(), Command::Append(5)),
        ]);
        assert_eq!(output[0], "HELLO");
        assert_eq!(output[1], "all roads lead to rome!");
        assert_eq!(output[2], "foobar");
        assert_eq!(output[3], "barbarbarbarbarbar");
    }
}
```

主要就是`transformer`方法里面，要做就是对传进来的 参数 进行操作， 参数是 一个Vector，里面的元素是一个元组类型

对于每一个元组，对第一个元素做相应的操作，这个操作由第二个元素给定。 第二个元素这里是一个枚举类型，列举出了三种操作

1. 变成大写
2. 去掉空白
3. 追加bar字符串，根据枚举对象关联的值，是几，就追加几个bar


## input类型和返回值类型

由测试案例得知

` pub fn transformer(input: Vec<String,Command>) -> Vec<String> {`

`assert_eq!(output[0],"HELLO")`这里跟取出来的对比不是 `&str` 吗？

是的，但是其实这里会做一件事就是 deref，这里是自动的

### 返回 `Vec<&str>` ???

可以这样，但是有点麻烦， 一方面是值的问题，另一方面那就涉及到生命周期了

我现在还没学到那么深，先粗略的了解一下，

因为这里给 `Vec` 里面塞了 `&str`类型，其实就是引用，而这个值肯定来子函数内部产生，因为Rust的特性，这个值在函数结束后肯定会释放了。

如果还想使用的话，要么就通过所有权转移出去，要么就需要是 static 的（不在堆上）

我感觉现阶段就先到这，还是先用 `Vec<String>`就行了

## `input.iter(c)`

其实

`for i in a` = `for i in a.into_iter()`

`for i in &a` = `for i in a.iter()`

`for i in &mut a` = `for i in a.iter_mut()`

对于 `input.iter()`,在[for and iterators](https://rustwiki.org/en/rust-by-example/flow_control/for.html#:~:text=This%20borrows%20each%20element%20of%20the%20collection%20through%20each%20iteration) 中介绍

> iter - This borrows each element of the collection through each iteration.

所以如果我们 input 定义的是 `Vec<(String,Command)>`

那么在这里  `for (string: &String,command: &Command) in input.iter()` 就应该是这样


这里我是这样写的
```rust
    pub fn transformer(input: Vec<(String, Command)>) -> Vec<String> {
        // TODO: Complete the output declaration!
        let mut output: Vec<String> = vec![];
        for (string, command) in input.iter() {
            // TODO: Complete the function body. You can do it!
            match command {
                &Command::Uppercase => output.push(string.to_uppercase().into()),
                &Command::Trim => output.push(string.trim().into()),
                &Command::Append(count) => {
                    let mut string = string.clone();
                    for _ in 0..count as i32 {
                        string = string + "bar";
                    }
                    output.push(string.into());
                }
            }
        }
        output
    }
```


### to_uppercase,trim

因为string是 `&String` 类型，其实在这里也可以理解为借用。所以直接用这个方法也可以

但是`to_uppercase()` 返回的是`String`类型

`trim()`返回的是`&str`类型

这里统统用一个 `into()` 交给编译去处理好了，我目前对他的理解是，他会根据上下文自动转化的


### match command or match *command

都可以,但是对应不同的用法

```rust
            match command {
                Command::Uppercase => output.push(string.to_uppercase().into()),
                Command::Trim => output.push(string.trim().into()),
                Command::Append(count) => {
                    let mut string = string.clone();
                    for _ in 0..*count as i32 {
                        string = string + "bar";
                    }
                    output.push(string.into());
                }
            }

```
注意 `*count as i32`的 * 号

```rust
            match *command {
                Command::Uppercase => output.push(string.to_uppercase().into()),
                Command::Trim => output.push(string.trim().into()),
                Command::Append(count) => {
                    let mut string = string.clone();
                    for _ in 0..count as i32 {
                        string = string + "bar";
                    }
                    output.push(string.into());
                }
            }
```

注意 `match *command`

```rust
            match command {
                &Command::Uppercase => output.push(string.to_uppercase().into()),
                &Command::Trim => output.push(string.trim().into()),
                &Command::Append(count) => {
                    let mut string = string.clone();
                    for _ in 0..*count as i32 {
                        string = string + "bar";
                    }
                    output.push(string.into());
                }
            }
```

注意 每一个分支前面的 `&`

### count as i32

因为` Command::Append(count) => {` 我这里解构出来的count其实是 `usize` 类型，枚举中定义的

然后 0..count这种形式需要 count是一个 integer



## `use` module

```rust
#[cfg(test)]
mod tests {
    // TODO: What do we need to import to have `transformer` in scope?
    use ???;
    use super::Command;
```

这里就是需要用 `use` 来创建一个 快捷键， 看下面直接用的是 `transformer`,所以引入进来就可以了

```rust
#[cfg(test)]
mod tests {
    // TODO: What do we need to import to have `transformer` in scope?
    use super::my_module::transformer;
    use super::Command;
```


## 其他人的写法

```rust
mod my_module {
    use super::Command;

    // TODO: Complete the function signature!
    pub fn transformer(input: Vec<(String, Command)>) -> Vec<String> {
        // TODO: Complete the output declaration!
        let mut output: Vec<String> = vec![];
        for (string, command) in input.iter() {
            // TODO: Complete the function body. You can do it!
            let s = match command {
                Command::Uppercase => string.to_uppercase(),
                Command::Trim => string.trim().into(),
                Command::Append(i) => string.to_owned() + &"bar".repeat(*i),
            };
            output.push(s);
        }
        output
    }
}
```
### to_owned方法

之所以要用这个方法是因为 string是 &String类型

字符串的 + 其实是需要 `String + &str -> String`

### repeat()方法

库函数不熟悉，学到了。
