---
title: ⚠️ 不是所有"嵌套 for 循环"复杂度都等于 O(n²)
layout: post
author: cyven
tags: algorithm complexity
categories: CS CS::Core
---



## 直觉: 两个for循环嵌套复杂度就是 O(n²)

```
for x in setA:
   for Y in setB:

```

## 纠错:去真正推导这次具体的循环到底在做什么

比如这个场景,我使用而为链表,来记录跨目录选中的文件, 第一维度链表是文件的目录, 第二维度的链表记录的是文件名

此时如果查找一个文件是否是选中, 此时需要遍历这个二维链表, 那么如果分析这个遍历算法的复杂度是多少。

具体分析一下一次查找操作,要走的节点数,最坏情况是,外层节点+内层节点.

虽然这里也涉及到两个for循环嵌套，但是 *外层是"一旦匹配就停下来，不会对每个目录都完整跑一遍内层"*

代码层面的表示

```
for dir in set(dirs):
	if match(dir,target_dir):
		for file in file_set(dir):
			if match_file(file,target_file):
				do_something()
```


这里形式上虽然也是两个for循环嵌套,但是其实是由匹配条件的.

*所以不是无条件的对每一个外层都执行一遍内层*, 那样则是O(n²),比如冒泡排序
