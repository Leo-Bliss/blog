---
title: "c++多线程总结"
date: 2022-08-31
description: ""
categories: [
	"C++",
	"多线程",
]	
tags: [
   
]
draft: false
---
## thread

基础概念：

+ 并发：一个处理器在一定时间间隔(时间片)内轮流执行任务。 多任务是抢占资源的。

+ 并行：多个处理器同一时间点上同时执行任务。 不抢占资源。
+ 临界资源： 在一段时间内仅允许一个进程访问的资源。比如打印机，各个进程采用互斥的方式实现对资源的共享，
+ 临界区： 每个进程访问临界资源的那段代码和共享变量有关的程序段。

### 并发的实现方式

+ 通过多个进程实现并发
+ 通过创建多个线程实现并发

#### 进程间通信

+ 同一电脑： 管道，共享内存，文件， 消息队列
+ 不同电脑： socket通信

#### 线程间通信

+ 共享内存，比如使用全局变量
+ 共享数据分析：
  + 只读数据：它是安全的稳定的，直接读即可；
  + 有读有写：不安全，最简单处理：读写互斥

#### 死锁

> 只有至少有两把锁才可能出现死锁

+ 描述：1持有A，想要B；2持有B，想要A。导致1,2无法继续执行的僵持状态

+ 互斥量：用于多线程编程种保护共享数据

+ 解决死锁：

  + 多个互斥量，保存相同的上锁顺序

  + 当多个互斥量时，使用`std::lock()`进行上锁，这个接口使得上锁顺序无关：`std::lock(mtx1, mtx2)`, 还是记住需要手动解锁， 可配合`std::lock_guard`避免手动解锁：

    ```cpp
    std::lock(m1, m2);  // 一次性锁多个
    std::lock_guard<std::mutex>(m1, std::adopt_lock);
    std::lock_guard<std::mutex>(m2, std::adopt_lock);
    ```

    

### API

```cpp
thread t(func); // 传入一个可调用对象: 函数对象，lambda函数， 仿函数等
t.join(); // 阻塞主线程 等子线程执行完后与主线程汇合
t.deatch(); // 分离，子线程执行不可控，子线程和主线程各自执行各的，主线程不需要等待， 子线程执行完毕有接管他的运行时库进行清理
```

+ 一般不需要关心或者控制子线程的执行时就调用`detach`
+ 使用`detach`时，需要避免线程入口函数的传参隐式类型的转换，防止局部对象失效， 可以显示构造一个临时对象传入避免不安全使用对象
+ 尽可能不使用`detach`
+ 明确往线程对象入口函数参数指定需要传入**真引用**则必须使用`std::ref`，否则都是默认拷贝，但是传真引用只有使用`join`才是安全的

#### lock_guard

+ `std::lock_gurad<std::mutex> lock(mtx, std::adopt_lock)`, `std::adopt_lock`适应之前mtx已经了lock，从而不再lock

#### unique_lock

+ 可以完全取代`lock_guard`, 而且更灵活，但是占有更多空间，相比`std::adopt_lock`还有其他参数
+ `std::try_to_lock`:尝试用`mutex`的`lock()`去锁定`mutex`， 如果没有锁成功也立即返回不会阻塞，但需要注意std::try_to_lock之前不能手动调用`mutex`的`lock()`否则会卡死。`unique_lock`对象成员函数`owns_lock() `可以 判断有没有拿到锁

+ `std::defer_lock`: 初始化一个没有加锁的`mutex`， 使用前提也是不能对传入的`mutex`对象`lock()`，否则报异常。
+ 成员函数：
  + `lock()`
  + `unlock`()：虽然会自动加锁解锁，但是提供lock和unlock是为了临时解锁处理一些非共享代码
  + `try_lock`(): return true or false
  + `release()`:释放管理的`mutex`的所有权，返回之前管理的`mutex`对象的指针，之后需要我们自己负责`mutex`对象的unlock
+ `unique_lock`对象所有权转移方法：使用std::move； 通过一个函数作为临时对象返回；

#### 条件变量

+ `std::condition_variable`:需要结合互斥量进行使用
+ `wait`():
  + 第一个参数为互斥量对象
  + 没有第二个参数，则解锁互斥量，当前线程执行阻塞到本行，直到其他线程notify_one；
  + 如果第二个参数为lambda，若lambda返回true则wait直接返回继续执行，否则解锁互斥量，阻塞到本行，直到被notify_one
  + 当被唤醒时： 重新尝试对互斥量加锁，直到加锁成功后继续往下执行
  + 虚假唤醒：即使没有notify该线程也有概率会唤醒，处理：wait中要第二个参数（lambda）并正确判断要处理的共享数据是否存在
  + 有可能出现一个线程还没走到wait时，notify消息就发送了，此时后面就可能导致盲目等待了，所以执行wait需要判断一些条件

  + `notify_one`():尝试把wait的线程唤醒

### future

+ `vaild()`: 判断值是否有效

+ `share()`: 用于future对象构造shared_future，比如`std::shared_future sf(f.share());`

+ future_status

  + ```cpp
    pass
    ```

+ `shared_future`: 用于多个线程获取future数据，由于普通future对象只支持get一次，`std::shared_future<int> sf = std::thread([]{return 1;});`

#### atomic

+ 原子操作概念：不可分割的操作，要么完成要么未完成， 不可能出现中间状态。在多线程种不会被打断的程序执行片段。 效率上比互斥量更高。适用于一个变量而不是一个代码段。能用原子操作就不用互斥量；一般针对`++, --, +=, -=`操作符，其他的不一定支持原子操作，比如`x = x + 1`不支持原子操作。
+ `load()`原子方式读取值，没有拷贝构造和赋值操作符， 可以这样进行复制：`std::atomic<int> m2(m1.load())`;
+ `store(v)`原子方式写入值，`std::atomic<int>m1 = 0; m1.store(1);`

#### async

> 用于创建异步任务

+ 第一个标志参数指定`std::launch::async`：**强制**异步任务在**新线程中执行**， 当future对象调用get或者wait时执行

+ 当第一个标志参数使用`std::launch::deferred`：不会创建新线程，会到主线程中执行，如果future没有get或者wait将不执行

+ 当第一个标志参数为`std::launch::async | std::launch::deferred `,**由系统评估选择其一执行**，当第一个参数不传入标志参数时，**默认就是这种参数**。

+ 当系统资源紧张的时候，使用`std::thread`可能创建线程失败，出现程序奔溃

+ 相比`std::thread`,`std::async`获取入口参数返回值更简单，由于可以默认由系统决定是否创建新线程，在资源紧张情况下，这种方式也更安全

+ 经验： 一个程序不宜超过100-200个线程

+ 当为默认标志参数时，系统采用的策略（`std::launch::async` ，`std::launch::deferred`）确定：

  ```cpp
  std::future<int> ret = std::async([]() { 
  			std::this_thread::sleep_for(std::chrono::milliseconds(5000));
  			std::cout <<" async this_thread id " << std::this_thread::get_id() << std::endl;
  			return 1024;});
  std::future_status status = ret.wait_for(std::chrono::seconds(0)); 
  if (status == std::future_status::deferred)
  {
      // 延迟执行，由于资源紧张没有创建新线程，将在主线程中执行
      std::cout << "deferred" << std::endl;
      //这个时候在主线程中执行
      std::cout << ret.get() << std::endl; 
  }
  else // 创建了新线程执行
  {
      if (status == std::future_status::ready)
      {
          // 线程执行完毕，成功返回
          std::cout << "ready" << std::endl;
          std::cout << ret.get() << std::endl;
      }
      else if (status == std::future_status::timeout)
      {
          // 线程还在执行
          std::cout << "timeout" << std::endl;
          std::cout << ret.get() << std::endl;
      }
  }
  
  ```
  
#### windows下的临界区
  
```cpp
  #include <windows.h>
  #include <vector>
  
  class A
  {
      public:
      	A()
          {
              InitializeCriticalSection(&_wCS);
          }
      	CRITICAL_SECTION _wCS; // 相当于std::mutex
      	void pushValue(int v)
          {
              EnterCriticalSection(&_wCS); // 相当于lock
              //EnterCriticalSection(&_wCS);
              vec.push_back(v);
              //LeaveCriticalSection(&_wCS);
              LeaveCriticalSection(&_wCS); // unlock
          }
    	private:
      	std::vector<int> vec;
  };
  
  class WindowsLockGuard
  {
      public:
      	WindowsLockGuard(CRITICAL_SECTION* wcs):_pCS(wcs){
              EnterCriticalSection(_pCS);
          }
      	~WindowsLockGuard()
          {
              LeaveCriticalSection(_pCS);
          }
      private:
      	CRITICAL_SECTION* _pCS;
  };
  ```
  
+ windows中允许在同一个线程中（不同线程会卡住等待）进入同一个临界区多次, c++11不允许在同一线程中lock普通互斥量多次，否则报异常；

#### recursive_mutex

+ 允许在同一线程中同一个递归互斥量多次lock
+ 一般使用它是代码设计的不太好，使用后需要考虑是否有优化空间，使得只lock一次

#### 带超时的互斥量

```cpp
	std::timed_mutex t_mtx; 
	std::recursive_timed_mutex rt_mtx; 
	if (t_mtx.try_lock_for(std::chrono::seconds(2))) // 尝试等待2s 获取锁
	{
		// 保护内容
		t_mtx.unlock();
	}
	else // 普通mutex没拿到锁会阻塞，没法走下这里
	{
		// 没有拿到锁
		// 可以处理不需要保护内容
	}

	// 直到某个时间点获取锁
	if (t_mtx.try_lock_until(std::chrono::steady_clock::now() + std::chrono::milliseconds(1000)))
	{

	}
	else
	{

	}
```

