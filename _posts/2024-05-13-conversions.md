---
title: 🧩 rustlings conversions
layout: post
author: cyven
tags: rust conversion from from_into as
categories: CS CS::Lang CS::Lang::Rust
---


## from_into

这个题就是要去完成 `impl From<&str> for Person {}` 方法

我自己的解法，就是很不rust

```rust
impl From<&str> for Person{
	fn from(s: &str) -> Person{
		let fields: Vec<&str> = s.split(',').collect();
		if fields.len() == 2{
			let name = fields.[0];
			let age = fields.[1];

			if let Ok(age) = age.parse::<usize>(){
				if name.len!=0{
					return Person{name:name,age:age};
				}
			}
		}

		Person::default()
	}

}
```

然后我看了其他人的写法，用了 `split_once` 和 `match` 风格更rust一点

```rust
impl From<&str> for Person {
    fn from(s: &str) -> Person {
        let (name, age) = match s.split_once(',') {
            Some((name, age)) => (name.trim(), age.trim()),
            None => return Person::default(),
        };
        if let Ok(age) = age.parse::<usize>() {
            if name.len() > 0 {
                return Person {
                    name: name.to_string(),
                    age: age,
                };
            }
        };
        Person::default()
    }
}
```


## from_str

感觉自己对match的用法还不够理解深入，光能识别，但用不起来，问题处在哪呢？

这是我写的解法

```rust

impl FromStr for Person {
    type Err = ParsePersonError;
    fn from_str(s: &str) -> Result<Person, Self::Err> {
        if s.is_empty() {
            return Err(ParsePersonError::Empty);
        };
        let fields: Vec<&str> = s.split(',').collect();
        if fields.len() != 2 {
            return Err(ParsePersonError::BadLen);
        } else {
            let (name, age) = (fields[0], fields[1]);
            if name.len() == 0 {
                return Err(ParsePersonError::NoName);
            }

            match age.parse::<usize>() {
                Ok(age) => {
                    return Ok(Person {
                        name: name.to_string(),
                        age: age,
                    });
                }
                Err(e) => return Err(ParsePersonError::ParseInt(e)),
            }
        }
    }
}
```

if，else 一大堆

看了别人的写法，最主要就是match的用法和 Error Propagation

```rust

impl FromStr for Person {
    type Err = ParsePersonError;
    fn from_str(s: &str) -> Result<Person, Self::Err> {
        if s.is_empty() {
            return Err(ParsePersonError::Empty);
        }

        let fields: Vec<&str> = s.split(',').collect();

        let (name, age) = match &fields[..] {
            [name, age] => (
                name.to_string(),
                age.parse::<usize>().map_err(ParsePersonError::ParseInt)?,
            ),
            _ => return Err(ParsePersonError::BadLen),
        };
        if name.is_empty() {
            return Err(ParsePersonError::NoName);
        }
        Ok(Person {
            name: name,
            age: age,
        })
    }
}

```

为什么 `ParsePersonError::ParseInt` 可以当作参数传给 `map_err` 呢？

因为在这里 `ParseInt` 其实是一个 tuple variant constructor，也就是是一个构造器，所以他是一个Fn，自然也是一个 FnOnce


然后这里match的其实是变量名


