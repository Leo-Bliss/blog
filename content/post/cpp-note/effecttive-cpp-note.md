---
title: "《More Effective C++》技术章节笔记"
date: 2022-08-31
description: "阅读《More Effective C++》技术章节过程中的总结"
categories: [
	"读书笔记",
	"C++"
]	
tags: [
    "C++",
]
draft: false
---

+ 不可真`virtuel`的函数可以借助可真`virtuel`的成员函数达到**伪**`virtual`。比如构造函数和非成员函数的"虚"化：

  + ```cpp
    class Component
    {
    public:
    	virtual std::ostream& print(std::ostream& out) const = 0;
    };
    
    class AComponent :public Component
    {
    public:
    	virtual std::ostream& print(std::ostream& out) const
    	{
    		out << "A";
    		return out;
    	}
    };
    
    class BComponent :public Component
    {
    public:
    	virtual std::ostream& print(std::ostream& out) const
    	{
    		out << "B";
    		return out;
    	}
    };
    
    std::ostream& operator<<(std::ostream& out, Component& c)
    {
    	return c.print(out);
    }
    
    // 使用
    AComponent a;
    BComponent b;
    std::cout << a << " " << b << std::endl;
    ```

  + 当没有`namespace`限定作用域时，类中friend函数为全局作用域，而类中static作用域限定在类空间

+ 多用`namespace`, 可以避免命名冲突，同时某些时候可以简化书写：

  + ```cpp
    namespace PrintingStuff
    {
    	class Printer
    	{
    	private:
    		Printer() {};
    		//...
    	public:
    		void doX() {};
    		void doY() {};
    		//...
    		friend Printer& thePrinter();
    	};
    
    	Printer& thePrinter()
    	{
    		static Printer printer;
    		return printer;
    	}
    }
    
    // 使用
    using PrintingStuff::thePrinter;
    thePrinter().doX();
    thePrinter().doY();
    ```

+ **不要**将**含`local static`对象**的**非成员函数`inline`化**：由于`inline`函数复制可能导致产生多个`static`对象副本

+ 避免具体类继承具体类

+ pass

+ 实现`String`类：引用计数，写时复制

  + ```cpp
    // version 1
    
    // my_string.h
    #pragma once
    #include <cstring>
    #include <iostream>
    
    namespace MyStr
    {
    	class String
    	{
    	public:
    		String(const char* str="");
    		String(const String& s);
    		String& operator=(const String& s);
    		~String();
    		const char& operator[](size_t index) const;
    		char& operator[](size_t index); // 重载实现写时复制
    		friend std::ostream& operator<<(std::ostream& out, String& s);
    	private:
    		struct StringValue
    		{
    			char* m_data;
    			int m_refCount;
    			StringValue(const char* str) :m_refCount(1) {
    				size_t len = std::strlen(str);
    				m_data = new char[len + 1];
    				std::strncpy(m_data, str, len);
    				m_data[len] = '\0';
    			};
    			~StringValue()
    			{
    				delete[] m_data;
    			}
    		};
    		StringValue* m_pValue;
    	};
    }
    
    // my_string.cpp
    #include "my_string.h"
    namespace MyStr
    {
    	String::String(const char* str):m_pValue(new StringValue(str))
    	{
    
    	}
    
    	String::String(const String& s)
    	{
    		m_pValue = s.m_pValue;
    		++m_pValue->m_refCount;
    	}
    
    	String& String::operator=(const String& s)
    	{
    		if (m_pValue == s.m_pValue)
    		{
    			return *this;
    		}
    		if (--m_pValue->m_refCount == 0)
    		{
    			delete m_pValue;
    		}
    		m_pValue = s.m_pValue;
    		++m_pValue->m_refCount;
    		return *this;
    	}
    
    	String::~String()
    	{
    		if (--m_pValue->m_refCount == 0)
    		{
    			delete m_pValue;
    		}
    	}
    
    	const char& String::operator[](size_t index) const
    	{
    		return m_pValue->m_data[index];
    	}
    
    	char& String::operator[](size_t index)
    	{
    		if (m_pValue->m_refCount > 1)
    		{
    			--m_pValue->m_refCount;
    			m_pValue = new StringValue(m_pValue->m_data);
    		}
    		return m_pValue->m_data[index];
    	}
    
    	std::ostream& operator<<(std::ostream& out, String& s)
    	{
    		out << s.m_pValue->m_data;
    		return out;
    	}
    }
    
    
    
    
    
    ```

  + 

