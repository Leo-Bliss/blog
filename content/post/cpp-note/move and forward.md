---
title: "move and forward"
date: 2022-09-06T23:50:00+08:00
description: "移动语义和完美转发总结"
categories: [
	"C++"
]	
tags: [
   "C++"
]
draft: false
---

# 移动语义和完美转发

+ 左值：表达式结束后依然存在的持久化对象，注意：具名右值引用是左值：`int&& val = 1024; // val为左值`

+ 右值：表达式结束后将不存在的临时对象

+ 右值引用：用于延长右值的生命周期

+ `std::move(v)`: 强制将左值转换为右值；

  + 本质**仅仅是强制类型转换**：`static_cast<typename std::remove_reference<T>::type&&>(t)`，使得可匹配右值形参函数；
  + **只有结合移动构造函数或移动赋值操作符才能起到减少不必要拷贝的意义**
  + 基本类型执行std::move并不会将原来变量内容改变,只是执行简单的拷贝，比如`int x = 1024; int y = std::move(x); //x依然为1024， y:1024`。
  + `const`左值引用可以引用右值(`const` 引用可以绑定非左值)： `const int & a = std::move(1)`

+ `std::forward<T>(v)` ：保持v原本的类型进行传递

  + 一般就用于模板中，功能表现上等价于`std::static_cast<T&&>`, （`T&&`基于引用折叠规则，也称万能引用）

  + **普通的传参转发不管传入左值还是右值，转发时都表现为左值**，此时就有了完美转发的用武之地了

  + 引用折叠规则（**这个规则只会出现在模板中**）： 只有**两个都是右值引用**互相折叠才能折叠为右值引用（&& && => &&），否则折叠都是左值引用 (&& &=>&, & &&=>&, & &=> &)

  + ```cpp
    void ref(int& v)
    {
        std::cout << "lvalue ref " << v << std::endl;
    }
    void ref(int&& v)
    {
        std::cout << "rvalue ref " << v << std::endl;
    }
    void pass(int& v)
    {
        ref(v); // 调用ref(int&)
        ref(std::move(v));// 调用ref(int&&)
        // 下面的这两种写法在这里没啥意义，所以std::forward多用于函数模板
        ref(std::forward<int&>(v)); //  调用ref(int&)
        ref(static_cast<int&>(v)); // 调用ref(int&)
        // 用于体会std::forward功能表现
        ref(std::forward<int&&>(v)); //  调用ref(int&&)
        ref(static_cast<int&&>(v)); // 调用ref(int&&)
    }
    void pass(int&& v)
    {
        ref(v); // 转发调用的是ref(int&)!!!
        
        // 下面的调用看起来怪怪的，但是只有这样才能保持v的原来类型
        ref(std::move(v));// 调用ref(int&&)
        ref(std::forward<int&&>(v)); // 按右值进行转发, 调用ref(int&&)
        ref(static_cast<int&&>(v)); // 调用ref(int&&)
       
    }
    
    // 调用
    pass(1024); 
    int v = 1024;
    pass(v); 
    ```

    上面的两个pass函数，不管去掉哪一个都会导致其中一种类型的传参失败，但是这样**写起来太繁琐了**，使用模板函数和万能引用`T&&`可以解决这个问题：

    ```cpp
    template<typename T>
    void pass(T&& v)
    {
        std::cout << " normal param passing: ";
        ref(v);
        std::cout << " std::move param passing: ";
        ref(std::move(v));
        std::cout << " std::forward param passing: ";
        ref(std::forward<T>(v));
        std::cout << " static_cast<T&&> param passing: ";
        ref(static_cast<T&&>(v));
    }
    // 调用
    pass(1024);  // pass(int&&)
    int v = 1024;
    pass(v); // pass(int&)
    ```

    

    


