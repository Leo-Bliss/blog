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

## ## 0x01

> 在<<Effective STL>>条目18中曾提到要避免使用`vector<bool>` 

+ `vector<bool>`不能算标准容器，标准容器要满足： `T* p = &vec[0]`可编译, 而它不满足：

  ```cpp
  	vector<bool> vec(100);
  	bool* bp = &vec[0]; // error, 提示类型对不上
      bool& br = vec[0]; // error
  ```

+ `vector<bool>`考虑到空间上的优化，它存的不是直接存的`bool`类型，而是一个个bit，类似`bitset`那种。

+ `[]`返回的是`std::vector< bool>:reference`, 导致当`auto a = vec[0];`， 修改`a`时会影响容器中的内容（这个需要特别小心）：

  ```
  	auto a = vec[0];
  	std::cout << vec[0] << endl; // 0
  	a = 1;
  	cout << vec[0] << endl; // 1 
  ```

+ `bool b = vec[0];`没问题是由于有隐式类型转换。