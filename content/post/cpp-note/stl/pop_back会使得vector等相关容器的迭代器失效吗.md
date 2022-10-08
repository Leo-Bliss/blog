---
title: "pop_back会使得vector等相关容器的迭代器失效吗？"
date: 2022-09-27T20:16:00+08:00
description: ""
categories: [
	
]	
tags: [
   
]
draft: true
---



## STL中哪些有pop_back这个成员函数

+ vector
+ string
+ deque
+ list

## pop_back对迭代器的影响

+ vector: 
  + `erase`: 在删除点或之后使迭代器和引用无效，包括 `end()` 迭代器
  + `push_back`:如果新size大于`capacity`所有迭代器和引用均失效，否则只有之前的`end`迭代器失效
  + `pop_back`: 最后一个元素的引用和迭代器， `end()`(它是最后一个元素下一个位置的迭代器)均会失效
+ string: cpp reference中没有提到，自己待验证。
+ deque:
  + `push_back`:所有迭代器包括之前的end迭代器均失效， 所有引用都不会失效。
  + `pop_back`:被删除元素的迭代器和引用失效，之前的end迭代器也失效。
  + `pop_front`:被删除元素的迭代器和引用失效，如果这个被删除元素是最后一个元素那么之前的end迭代器也失效。
+ list:
  + `erase`: 只有被删除的元素迭代器和引用会失效，注意传入的迭代器要求有效且可解引用，所以不可以是`end`,因为它不可以解引用
  + `push_back`: 没有引用和迭代器会失效
  + `pop_back`: 被删除的元素迭代器和引用失效
  + `pop_front`:被删除的元素迭代器和引用失效



