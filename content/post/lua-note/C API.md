---
title: "lua-C API"
date: 2021-07-25
description: "lua 相关的c api"
categories: [
	"Lua",
]	
tags: [
   
]
draft: false
---

## 一个简单的lua 解释器
```c++
// simpleLuaInterpreter.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

#include <iostream>
#include <cstring>
#ifdef __cplusplus
extern "C" {
#endif
#include "lua.h" // 高度抽象化的基础API
#include "lauxlib.h" // 包含辅助库函数声明，函数以luaL_开头
#include "lualib.h" // 包含打开库的函数的声明
#ifndef __cpluscplus
}
#endif // !__cpluscplus

// 辅助库没有方式去解释lua,只能通过lua.h中的基础API
/*

Lua库根本没有定义C的全局变量,它将其所有状态保持在动态结构中。

Lua内部的所有函数都会收到指向该结构的指针。
*/
// #include "lua.hpp" //可代替extern "C"{#include "lua.h"}

int main()
{
    char buff[256];
    int error;
    lua_State* L = luaL_newstate(); // 创建一个新状态但是它的环境中没有预先定义任何函数
    luaL_openlibs(L); // 打开所有标准库
    // luaL_loadstring:编译用户输入，如果不出错，return0并将结果压入stack
    // lua_pcall 不出错return 0
    // 两个函数出错会将错误信息压入stack
    while (fgets(buff, sizeof(buff), stdin) != NULL) {
        error = luaL_loadstring(L, buff) || lua_pcall(L, 0, 0, 0);
        if (error) {
            fprintf(stderr, "%s\n", lua_tostring(L, -1));
            lua_pop(L, 1);
        }
        lua_close(L);

    }
    std::cout << "Hello World!\n";
    return 0;
}

```


## lua 和 C 组件的通信 
lua 和 C 主要组件的通信 是通过强大的**虚栈（virtual stack）**
![image-20210808235705964.png](https://cdn.nlark.com/yuque/0/2021/png/22158835/1628438458990-3d4e7c56-3359-476d-8d94-679e5f47bfe9.png#clientId=u5dd5abf4-1009-4&from=paste&height=294&id=u3f3da6a6&margin=%5Bobject%20Object%5D&name=image-20210808235705964.png&originHeight=587&originWidth=964&originalType=binary&ratio=1&size=25992&status=done&style=none&taskId=u50c835f9-6a2d-4c0b-95ab-c5cf8352bc4&width=482)

### 虚栈压入元素
```
 void lua_pushnil (lua_State *L);
 void lua_pushboolean (lua_State *L, int bool);
 void lua_pushnumber (lua_State *L, lua_Number n);
 void lua_pushinteger (lua_State *L, lua_Integer n);
 void lua_pushlstring (lua_State *L, const char *s, size_t len);
 void lua_pushstring (lua_State *L, const char *s);
```
向栈中压入元素前要保证stack中至少有20个空位（slots）,在lua.h中定义了这个常量宏：LUA_MINSTACK；
检查虚栈空间是否足够的函数：

- int lua_checkstack (lua_State *L, int sz);
- void luaL_checkstack (lua_State *L, int sz, const char *msg);

参数sz是需要检查的slots数量，第二个函数使得可以返回一个自定义的出错信息而不是错误码；
### 虚栈提取元素

- index可正可负，第一个压入元素（栈底）的index=1,最后压入元素（栈顶）的index=-1；
- 获取栈中元素个数：size_t n = lua_gettop(L)；
- **检查获取元素的类型** 可以使用：int lua_is* (lua_State *L, int index);,*换成想要检查的**lua数据类型** ，比如lua_isnil,lua_isstring。lua_isnumber和lua_isstring比较特殊，两者并不能确定元素类型而是是检查index位置上的元素是否可以转换为该类型。除此之外可以使用lua_type(L,index)获取到元素类型，返回值是lua.h中定义的宏，比如：LUA_TNIL, LUA_TBOOLEAN, LUA_TNUMBER, LUA_TSTRING。
- 获取元素的函数：

 int lua_toboolean (lua_State *L, int index); // nil和false时为0，其他类型元素为1
 const char *lua_tolstring (lua_State *L, int index,size_t *len); // 同时获取字符串长度，存到len中
 const char *lua_tostring (lua_State *L, int index); // 不需要长度信息可使用这个函数获取字符串
 lua_State *lua_tothread (lua_State *L, int index); // index处获取一个线程，失败时返回NULL
 lua_Number lua_tonumber (lua_State *L, int index);
 lua_Integer lua_tointeger (lua_State *L, int index);

### 其他虚栈操作
int lua_gettop (lua_State *L); // 获取栈元素个数，也是栈顶的的正index
void lua_settop (lua_State *L, int index); // 当index大于现在栈元素个数时补nil,小于时丢弃多的，置空：lua_settop(L,0)
void lua_pushvalue (lua_State *L, int index); // 拷贝一个index处的元素压入栈中
void lua_rotate (lua_State *L, int index, int n); // 从index处到栈顶，旋转n个位置
void lua_remove (lua_State *L, int index); // 移除index处元素
void lua_insert (lua_State *L, int index); // 将index处元素放到栈顶
void lua_replace (lua_State *L, int index); // 弹出栈顶元素赋值到index处
void lua_copy (lua_State *L, int fromidx, int toidx);

- lua_settop (lua_State *L, int index)的index也可以是负下标，由此定义了一个弹出n个元素的宏#define lua_pop(L,n) lua_settop(L,-(n)-1)比如弹出3个元素，lua_pop(L,3) =>lua_settop(L,-4),栈顶-1设置成-4，由-1 to -4 弹出了3个元素；
- remove宏可通过rotate和pop来实现：

#define lua_remove(L,idx) (lua_rotate(L, (idx), -1), lua_pop(L, 1))

- insert 的宏实现#define lua_insert(L,idx) lua_rotate(L, (idx), 1)

![image-20210808235805302.png](https://cdn.nlark.com/yuque/0/2021/png/22158835/1628438498960-423715fd-ddbe-4469-be1d-f9e70c94cade.png#clientId=u5dd5abf4-1009-4&from=paste&height=402&id=u053dd0ca&margin=%5Bobject%20Object%5D&name=image-20210808235805302.png&originHeight=804&originWidth=969&originalType=binary&ratio=1&size=120930&status=done&style=none&taskId=u48dd8639-a6d2-4e23-8918-4f3b8c1a3cc&width=484.5)

- lua_rotate 源码：
```c
LUA_API void lua_rotate (lua_State *L, int idx, int n) {
  StkId p, t, m;
  lua_lock(L);
  t = L->top - 1;  /* end of stack segment being rotated */
  p = index2stack(L, idx);  /* start of segment */
  api_check(L, (n >= 0 ? n : -n) <= (t - p + 1), "invalid 'n'");
  m = (n >= 0 ? t - n : p - n - 1);  /* end of prefix */
  reverse(L, p, m);  /* reverse the prefix with length 'n' */
  reverse(L, m + 1, t);  /* reverse the suffix */
  reverse(L, p, t);  /* reverse the entire segment */
  lua_unlock(L);
}

```
### 测试代码
```c

#include <iostream>
#include <string>

extern "C" {
#include "lua.h" // 高度抽象化的基础API
#include "lauxlib.h" // 包含辅助库函数声明，函数以luaL_开头
#include "lualib.h" // 包含打开库的函数的声明
}


static void dumpStack(lua_State* L)
{
    size_t n = lua_gettop(L);
    for (size_t i = 1; i <= n; ++i) {
        int t = lua_type(L, i);
        switch (t)
        {
        case LUA_TBOOLEAN: {
            std::cout << (lua_toboolean(L, i) ? "true" : "false") << std::endl;
            break;
        }
        case LUA_TSTRING: {
            std::cout << (lua_tostring(L, i)) << std::endl;
            break;
        }
        case LUA_TNUMBER: {

            if (lua_isinteger(L, i)) {
                std::cout << lua_tointeger(L, i) << std::endl;
            }
            else {
                std::cout << lua_tonumber(L, i) << std::endl;
            }
            break;
        }
        default: {
            std::cout << lua_typename(L, i) << std::endl;
            break;
        }
        }
    }
}
class SmartLuaState {
public:
    SmartLuaState(lua_State* LL=nullptr):L(LL) {
    }
    ~SmartLuaState() {
        if (L != nullptr)
        {
            std::cout << "close lua_State" << std::endl;
            lua_close(L);
        }
    }
    void pushnil() 
    {
        lua_pushnil(L);
    }
    void pushboolean(int n) 
    {
        lua_pushboolean(L, n);
    }
    void pushstring(const char *s) 
    {
        lua_pushstring(L, s);
    }
    lua_State* get_state() {
        return L;
    }
    void dumpStack();
    void rotate(int index,int n)
    {
        lua_rotate(L, index, n);
    }
    void insert(int index) {
        lua_insert(L, index);
    }
private:
    lua_State* L;
};

void SmartLuaState:: dumpStack()
{
    std::cout << "values of stack" << std::endl;
    size_t n = lua_gettop(L);
    for (size_t i = 1; i <= n; ++i)
    {
        int t = lua_type(L, i);
        switch (t)
        {
        case LUA_TBOOLEAN: {
            std::cout << (lua_toboolean(L, i) ? "true" : "false") << std::endl;
            break;
        }
        case LUA_TSTRING: {
            std::cout << (lua_tostring(L, i)) << std::endl;
            break;
        }
        case LUA_TNUMBER: {

            if (lua_isinteger(L, i)) {
                std::cout << lua_tointeger(L, i) << std::endl;
            }
            else {
                std::cout << lua_tonumber(L, i) << std::endl;
            }
            break;
        }
        default: {
            std::cout << lua_typename(L, i) << std::endl;
            break;
        }
        }
    }
}
static void test1()
{
    SmartLuaState smt = SmartLuaState(luaL_newstate());
    lua_State* L = smt.get_state();
    lua_pushboolean(L, 1);
    lua_pushnumber(L, 10);
    lua_pushnil(L);
    lua_pushstring(L, "hello");
    dumpStack(L);
    std::cout << "--- hello lua -- " << std::endl;
    smt.dumpStack();  
}

static void test2() 
{
    SmartLuaState state = SmartLuaState(luaL_newstate());
    state.pushstring("a");
    state.pushstring("b");
    state.pushstring("c");
    state.pushstring("d");
    state.dumpStack();
    state.rotate(2, 4);
    //state.insert(2);
    state.dumpStack();
    //state.rotate(2, -1);   
}
int main()
{
    // test1();
    test2();
    return 0;
}


```



