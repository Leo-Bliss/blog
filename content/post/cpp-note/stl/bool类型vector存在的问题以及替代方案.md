---
title: "bool类型vector存在的问题以及替代方案"
date: 2022-09-23T21:21:00+08:00
description: "总结vector<bool>类型注意要点和一些替代方案"
categories: [
	“C++"
]	
tags: [
   "STL"
]
draft: false
---

## ## 0x01 问题分析

> 在<<Effective STL>>条目18中曾提到要避免使用`vector<bool>` 

### 分析

+ `vector<bool>`不能算标准容器，标准容器要满足： `T* p = &vec[0]`可编译, 而它不满足：

  ```cpp
  	vector<bool> vec(100， false);
  	bool* bp = &vec[0]; // error, 提示类型对不上
      bool& br = vec[0]; // error
  ```

+ `vector<bool>`考虑到空间上的优化，它存的不是直接存的`bool`类型，而是一个个bit，相当于可动态扩容的`bitset`。

+ `[]`返回的是一个`reference`, 导致当元素赋值给`auto`变量并存在修改`auto`变量时，会影响容器中的内容（这个需要特别小心）：

  ```
  	vector<bool> vec(100, false);
  	auto a = vec[0];
  	std::cout << boolalpha <<vec[0] << endl; // false
  	a = true;
  	std::cout << boolalpha << vec[0] << endl; // true
  ```

+ `bool b = vec[0];`没问题是由于有隐式类型转换。

### 小结

通过上面的分析总结`vector<bool>`存在两个小坑（问题）：

+ `bool`指针（引用）没法直接指向元素
+ 元素赋值给`auto`变量，修改该变量会影响容器中元素

## 0x02 替代方案

1. 不需要动态扩容可以考虑使用`array`  (PS:  `array`似乎被人遗忘了，刷题中上场率不高~):

    ```cpp
      array<bool, 100> vis;
      fill(vis.begin(), vis.end(), false);
    ```

2.  可以使用`vector<unsigned int>`:

   ```cpp
       using mbool = unsigned int;
       vector<mbool> vis(100, false);
       cout <<vis[0] << endl; // 0
       auto b = vis[0]; // 主要是这里 降低了误修改容器值的概率
       b = true;
       cout <<vis[0] << endl; // 0
   	// 由于不是真正的bool所以不支持<<boolalpha直接显示true,false
   ```

3. 使用`dequeue`：

   除了`reserver`(因为内存不必连续，它根本不需要这个操作)，基本上`vector`能做的操作它都可以做。

   ```cpp
   	int n;
   	cin >> n;
   	deque<bool> vis(n, false);
   	//vis.assign(100, false); // ok
   	cout << boolalpha << vis[0] << endl; // false
   	auto b = vis[0];
   	b = true;
   	cout << boolalpha << vis[0] << endl; // false
   	bool* bp = &vis[0]; // ok
   	bool& br = vis[1]; // ok
   	br = true;
   	cout << boolalpha << vis[1] << endl; // true
   	for (auto&& v : vis)
   	{
   		cout << boolalpha << v << " ";
   	}
   	cout << endl;
   	
   ```

   **如果真的有必要**，个人更倾向**第1种和第3种替代方案**， 第2种反直觉且不方便。

## 0x03 注意

+ 并不能**为了避免上面提到的`vector<bool>`存在的问题**而用`bitset`来替代， 因为前面已经提到`vector<bool>`相当于可动态扩容的`bitset`，而且`bitset`并没有基于范围的for循环哦：

   ```cpp
	bitset<100> bits;
   	bits.set(0);  // => bits[0] = true
   	cout << std::boolalpha<<bits[0] << endl;
   	bits.reset(0); // => bits[0] = false
   	cout << std::boolalpha << bits[0] << endl;
   	bits[1] = true;
   	cout << std::boolalpha << bits[1] << endl; // true
   	bits[1].flip();
   	cout << std::boolalpha << bits[1] << endl; // false
   	auto b = bits[1]; // reference
   	b = true;
   	cout << std::boolalpha << bits[1] << endl; // true
   	for (auto b : bits) // error
   	{
   
   	}
   ```
   
   + `bitset`可以方便操作bit位，它转换成`string` 和`unsigned long`的接口也很强大：

     + [to_string](https://en.cppreference.com/w/cpp/utility/bitset/to_string)
     + [to_ulong](https://en.cppreference.com/w/cpp/utility/bitset/to_ulong)
     + [to_ullong](https://en.cppreference.com/w/cpp/utility/bitset/to_ullong)
   
     

