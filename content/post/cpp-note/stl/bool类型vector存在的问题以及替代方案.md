---
title: "bool类型vector存在的问题以及替代方案"
date: 2022-09-23T21:21:00+08:00
description: "总结vector<bool>类型注意要点和一些替代方案"
categories: [
	
]	
tags: [
   
]
draft: false
---

## ## 0x01 问题分析

> 在<<Effective STL>>条目18中曾提到要避免使用`vector<bool>` 

+ `vector<bool>`不能算标准容器，标准容器要满足： `T* p = &vec[0]`可编译, 而它不满足：

  ```cpp
  	vector<bool> vec(100);
  	bool* bp = &vec[0]; // error, 提示类型对不上
      bool& br = vec[0]; // error
  ```

+ `vector<bool>`考虑到空间上的优化，它存的不是直接存的`bool`类型，而是一个个bit，相当于可动态扩容的`bitset`。

+ `[]`返回的是`std::vector< bool>:reference`, 导致当`auto a = vec[0];`， 修改`a`时会影响容器中的内容（这个需要特别小心）：

  ```
  	auto a = vec[0];
  	std::cout << vec[0] << endl; // 0
  	a = 1;
  	cout << vec[0] << endl; // 1 
  ```

+ `bool b = vec[0];`没问题是由于有隐式类型转换。

## 0x02 替代方案

+ 不需要动态扩容，固定大小:

  1. 可以考虑使用`array`  (PS:  `array`似乎被人遗忘了，刷题中上场率不高~):

      ```cpp
        array<bool, 100> vis;
        fill(vis.begin(), vis.end(), false);
      ```

  2.  可以使用`vector<unsigned int>`:

     ```cpp
     	using mbool = unsigned int;
     	vector<mbool> vis(100, false);
   	cout <<vis[0] << endl; // 0
     	auto b = vis[0];
     	b = true;
     	cout <<vis[0] << endl; // 0
     ```
  
     
  
+ 注意： 并不能为了避免`vector<bool>`存在的问题而用`bitset`来替代， 因为前面已经提到`vector<bool>`相当于动态的`bitset`, 

   而且`bitset`并没有基于范围的for循环哦：

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

   

