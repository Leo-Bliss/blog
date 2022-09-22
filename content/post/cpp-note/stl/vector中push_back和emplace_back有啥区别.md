---
title: "vector中push_back和emplace_back有啥区别？"
date: 2022-09-08T22:57:00+08:00
description: ""
categories: [
	"C++"
]	
tags: [
   "STL"
]
draft: false
---


## vector 

+ `push_back` 和`emplace_back`:  
  + 总结：

    + emplace_back支持直接传入对象初始化参数列表（由于c++11可变模版参数使得可以传入很多参数）
    并会在vector内存中原地构造， push_back支持传入一个对象参数，
    但是表现是先构造一个临时对象然后执行移动构造（下方test4中注释为运行结果）。
    + 其他用法两者没有区别

  + 测试代码
  ```cpp
    class Value
    {
    public:
    	Value(int x, int y = 2) :_x(x), _y(y)
    	{
    		std::cout << "Value()" << std::endl;
    	}
    	~Value()
    	{
    		std::cout << "~Value()" << std::endl;
    
    	}
    	Value(const Value& v)
    	{
    		std::cout << "Value(const Value&)" << std::endl;
    	}
    	Value(Value&& v) noexcept
    	{
    		std::cout << "Value(Value&&)" << std::endl;
    	}
    	Value& operator=(const Value& v)
    	{
    		std::cout << "operator=(const Value&)" << std::endl;
    	}
    	Value& operator=(Value&& v) noexcept
    	{
    		std::cout << "operator=(Value&&)" << std::endl;
    	}
    
    	int getX() const { return _x; }
    	int getY() const { return _y; }
    
    private:
    	int _x;
    	int _y;
    };
    
    void test1()
    {
    	std::vector<Value> vec;
    	vec.reserve(5);
    	Value a(0, 0);
    
    	std::cout << "## push_back val" << std::endl;
    	vec.push_back(a); // Value(const Value&)
    	std::cout << "## emplace_back val" << std::endl;
    	vec.emplace_back(a); // Value(const Value&)
    	std::cout << std::string(100, '-') << std::endl;
    }
    
    void test2()
    {
    	std::vector<Value> vec;
    	vec.reserve(5);
    	Value a(3, 4);
    	std::cout << "## push_back val" << std::endl;
    	vec.push_back(std::move(a)); // Value(Value&&)
    
    	std::cout << "## emplace_back val" << std::endl;
    	// 注意这里只是为了测试才再对同一对象std::move，
    	// 实际上如果移动构造出现资源转移等指针操作，再次std::move同一对象很危险
    	vec.emplace_back(std::move(a)); // Value(Value&&)
    	std::cout << std::string(100, '-') << std::endl;
    }
    
    void test3()
    {
    	std::vector<Value> vec;
    	vec.reserve(5);
    	Value a(0, 0);
    	std::cout << "## push_back val" << std::endl;
    	vec.push_back(Value(1)); // Value() Value(Value&&)
    
    	std::cout << "## emplace_back val" << std::endl;
    
    	vec.emplace_back(Value(1)); // Value() Value(Value&&)
    	std::cout << std::string(100, '-') << std::endl;
    }
    
    void test4()
    {
    	std::vector<Value> vec;
    	vec.reserve(5);
    
    	std::cout << "## push_back val" << std::endl;
    	//vec.push_back(1, 2); // error
    	vec.push_back(1); // Value() Value(Value&&)
    
    	std::cout << "## emplace_back val" << std::endl;
    
    	vec.emplace_back(1, 2); // Value()
    	std::cout << std::string(100, '-') << std::endl;
    }
    
  ```