---
title: rustlings hashmap3
layout: post
author: cyven
tags: rust
categories: CS CS::Lang CS::Lang::Rust
---



## Rustlings HashMap 3

这道题来自 rustling hashmap3

 A list of scores (one per line) of a soccer match is given. Each line is of
 the form : "<team_1_name>,<team_2_name>,<team_1_goals>,<team_2_goals>"
 Example: England,France,4,2 (England scored 4 goals, France 2).

 You have to build a scores table containing the name of the team, the total
 number of goals the team scored, and the total number of goals the team
 conceded. One approach to build the scores table is to use a Hashmap.
 The solution is partially written to use a Hashmap,
 complete it to pass the test.

 Make me pass the tests!



```rust

use std::collections::HashMap;

// A structure to store the goal details of a team.
struct Team {
    goals_scored: u8,
    goals_conceded: u8,
}

fn build_scores_table(results: String) -> HashMap<String, Team> {
    // The name of the team is the key and its associated struct is the value.
    let mut scores: HashMap<String, Team> = HashMap::new();

    for r in results.lines() {
        let v: Vec<&str> = r.split(',').collect();
        let team_1_name = v[0].to_string();
        let team_1_score: u8 = v[2].parse().unwrap();
        let team_2_name = v[1].to_string();
        let team_2_score: u8 = v[3].parse().unwrap();
        // TODO: Populate the scores table with details extracted from the
        // current line. Keep in mind that goals scored by team_1
        // will be the number of goals conceded by team_2, and similarly
        // goals scored by team_2 will be the number of goals conceded by
        // team_1.
    }
    scores
}
```

其实题目很简单，就是往一个HashMap里面插入值或者更新值

因为对每个 `r` 都要对 team1 和 team2 做一次操作，所以可以写一个函数干这件事

## 定义函数

```rust
fn update_one_team_score(scores:& mut HashMap<String,Team> ,team_name: String, team_score: u8,opponent_score: u8){
	/*
	 * Update two goals in the struct extracted from scores by team_name by adding team_score and opponent_score correspondingly.
	 *
	 */

}
```

这里的 `scores` 类型要 `& mut HashMap<String,Team>`  可变！

## 获取值

如果原来就已经有值了，要先取出来，如果没有就插入
可以用 `get` 方法，但因为涉及到修改，所以要用 `get_mut`方法

```rust
pub fn get_mut<Q>(&mut self, k: &Q) -> Option<&mut V>
where
    K: Borrow<Q>,
    Q: Hash + Eq + ?Sized,
```

因为返回的是 Option，所以这里可以用 模式匹配 来处理两种情况

```rust
fn update_one_team_score(scores:& mut HashMap<String,Team> ,team_name: String, team_score: u8,opponent_score: u8){
	/*
	 * Update two goals in the struct extracted from scores by team_name by adding team_score and opponent_score correspondingly.
	 *
	 */

	let t = scores.get_mut(&team_name);
	match t{
		Some(v) => {},

		None => {},
	}
}
```

如果`Some(v)` 匹配成功，那么t就不能用了， 因为部分所有权转移到了 v

## 取出结构体，并更新

`get_mut`方法返回的是 ` Option<&mut V>`
模式匹配得到的 v 则是 `&mut V` 类型

所以需要先解引用才能得到真正的结构体数据


```rust
fn update_one_team_score(scores:& mut HashMap<String,Team> ,team_name: String, team_score: u8,opponent_score: u8){
	/*
	 * Update two goals in the struct extracted from scores by team_name by adding team_score and opponent_score correspondingly.
	 *
	 */

	let t = scores.get_mut(&team_name);
	match t{
		Some(v) => {
			(*v).goals_scored += team_score;
			(*v).goals_conceded += opponent_score;
		},

		None => {},
	}
}
```

`*v.goals_scored` = `*(v.goals_scored)` 所以这不是我们希望的


## 没有值的话直接插入


```rust
fn update_one_team_score(scores:& mut HashMap<String,Team> ,team_name: String, team_score: u8,opponent_score: u8){
	/*
	 * Update two goals in the struct extracted from scores by team_name by adding team_score and opponent_score correspondingly.
	 *
	 */

	let mut t = scores.get_mut(&team_name);
	match t{
		Some(v) => {
			(*v).goals_scored += team_score;
			(*v).goals_conceded += opponent_score;
		},
		None => {
			scores.insert(team_name,Team{goals_scored:team_score,goals_conceded:opponent_score}); //这里别忘了加; 因为insert是有返回值的
		},
	}
}
```
上面那里 insert 语句 `;` 要加上，因为insert是有返回值，如果不加，它是一个表达式，而这个表达式的返回值会作为这个curly block的返回值，

**而 match的每一个分支是需要返回值相同的**

在Some分支中，其实返回的是 `()`,所以要求 `None` 分支也要返回这个，所以要加 `;`

加了 `;` 返回的默认值就是 `()`
