---
title: "vector几种初始化方式"
date: 2022-09-18T23:54:00+08:00
description: "vector几种常用初始化方式"
categories: [
	"C++"
]	
tags: [
   "STL"
]
draft: false
---
## 构造时初始化为指定值
```cpp
int n = 100;
std::vector<int> v(n); // 未指定值时，底层调用memset初始化n个0
std::vector<int> vec(n, 1);
```
## 成员函数assign
```cpp
// n = 100
std::vector<int> vec2;
vec2.assign(n, 0);// 如果n比原来capacity大，则重新分配更大的内存，然后初始化为指定的值
std::cout << vec.size() << " = " << vec.capacity() << std::endl; // 100, 100
vec.assign(50, 2); // new size < old size, 会resize，多的部分元素会调用析构函数
std::cout << vec.size() << " = " << vec.capacity() << std::endl; // 50, 50
```

## 借助填充算法
### 填充为相同的值

+ 基本用法
  + `std::fill(firstIterator, lastIterator, value)`
  + `std::fill_n(firstIterator, n, value)`
+ 定义在头文件`<algorithm>`中

+ 实例：
```cpp
fill(vec.begin(), vec.end(), 1);
fill_n(vec.begin(), vec.size(), 3);
```
### iota填充为一个序列
+ c++11提供的算法,定义在头文件`<numeric>`中
+ 定义：
```cpp
template< class ForwardIt, class T >
void iota( ForwardIt first, ForwardIt last, T value );
```
+ 实例：
```cpp
int startValue = 1;
std::iota(vec.begin(), vec.end(), startValue); // startValue, startValue+1...startValue+vec.size()
```






