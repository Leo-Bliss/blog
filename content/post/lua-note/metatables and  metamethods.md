---
title: "lua-metatables and  metamethods"
date: 2021-07-21
description: "lua元表和元方法"
categories: [
	"Lua",
]	
tags: [
   
]
draft: false
---

> metatables and  metamethods


- 元表让我们在面对一些未知操作时**改变表的一些行为**（通过 修改元表中的 元函数 实现运算符重载，控制table访问,修改键值操作...）；

- lua中每一个值**都可以有**一个**元表**，`table`和 `userdata`类型的值具有单个元表；其他类型的值为该类型的所有值共享一个元表；

- 但是在lua中只能修改 值类型 为`table`的元表，其他类型的元表修改需要通过C语言代码和debug库;

### 修改和获取元表

- `setmetatable(tb,mt)`：设置`tb`的元表为`mt`,并返回`tb`;
- `getmetatable(tb)`: 获取到`tb`的元表；
- 上面两个方法实质都是通过元表中的`__matetable`进行相关操作,通过`__matetable`重新赋值可以保证元表的安全，即阻止这两个函数对元表进行操作；
```lua
-- 修改元表
local t = {} -- 普通表
local mt = {} -- 元表：里面主要放一些元方法
setmetatable(t,mt) -- 设置 mt 为 t 的元表

-- 以上两条语句等价于
local mytable = setmetatable({},{})

 -- 获取元表
 print(getmetatable(t) == mt) -- true
 print(getmetatable({})) -- nil
 print(getmetatable("hello")) -- table: 007E9A60
 print(getmetatable("lua")) -- table: 007E9A60
 print(getmetatable(111)) -- nil
 
```
### 常见的元方法

1. 运算符元方法:
```lua
--[[
-- __add	  : '+'.
-- __sub	  : '-'.
-- __mul	  : '*'.
-- __div	  : '/'.
-- __mod	  : '%'.
-- __pow    : '^'
-- __unm	  : '-'.
-- __concat	: '..'.
-- __eq	    : '=='.
-- __lt	    : '<'.
-- __le	    : '<='
-- __band   ： '&'
-- __bor    : '|'
...
--]]
```

2. 其他元方法：
```lua
--[[
__tostring ： 对应于tostring
__index： 访问元素触发
__newindex: 增加元素触发
__call:对象当作函数一样调用时触发
--]]
```
### 元方法的使用
#### 运算符元方法和`__tostring`
> 通过编写运算符元方法实现运算符的重载

##### 
##### 编写简单的Set
```lua
-- D:\lua_note\chapter20\set.lua

local Set = {} -- 注意这个lua文件名不能和这个名字一样，否则出现未定义错误。
local mt = {} -- set的元表

-- 创建一个集合，集合特性：1. 确定性；2. 互异性；3. 无序性；
function Set.new(arr)
    -- arr = arr or {} -- 可对传的参数作缺省处理保证for迭代不出错
    local set = {}
    setmetatable(set,mt) -- 使得每一个新建的set有相同的元表
    for _,v in ipairs(arr) do -- 注意这里要记录的是v
        set[v] = true
    end
    return set
end

-- 集合并集
function Set.union(a,b)
    -- if getmetatable(a)~=mt or getmetatable(b)~=mt then
    --     error("matetable is mismathching")
    -- end
    local res = Set.new({}) -- 注意这里括号里要传一个table,否则new参数为nil无法构成迭代器而报错;或者new里对传参作缺省处理;
    for k in pairs(a) do
        res[k] = true
    end
    for k in pairs(b) do
        res[k] = true
    end
    return res
end

-- 集合交集
function Set.intersection(a,b)
    local res = Set.new({})
    for k in pairs(a) do
        res[k] = b[k]
    end
    return res
end

-- 转换为字符串
function Set.tostring(set)
    local list = {}
    for e,_ in pairs(set) do
        list[#list + 1] = tostring(e)
    end
    -- table.sort(list) -- 显示时可以排个序方便观察，只是显示并没有对set内部产生任何影响
    return "{" .. table.concat(list,',') .. "}"
end

-- 构建一个有序set
function Set.sort(set)
    local st = {}
    for k,_ in pairs(set) do
        st[#st + 1] = k
    end
    table.sort(st)
    return setmetatable(st,mt) -- 这里只能重新构建一个有相同元表的“类Set“对象才能保证是一个有序set
end

-- 为元表添加元方法
-- 方式1
mt.__add = Set.union

mt.__sub = Set.intersection

mt.__tostring = Set.tostring -- 使得使用print输出直接为我们Set.tostring中设定的格式，因为print会自动调用tostring;
-- 当使用io.write输出是需要显示调用tostring，即：io.write(tostring(set_obj))

-- 方式2
-- 子集：<= ，结合Set底层实现来编写比较逻辑
mt.__le = function (a,b)
    for k,_ in pairs(a) do
        if not b[k] then
            return false
        end
    end
    return true
end

-- 真子集：<
mt.__lt = function (a,b)
    return a<=b and not (b<=a)
end

-- 相等： ==
mt.__eq = function (a,b)
    return a<=b and b<=a
end

--[[为了保证元表的安全:
-- 即 不被getmetetable获取到真实元表 以及 setmatetable修改我们真实的元表，
-- 只需为__metatable赋一个值即可；
-- 即getmetetable和setmatetable获取和修改的只是元表中__metatable 的值;
-- 思考： 也就是说__metatable 存的是 元表本身（self）？
--]]

mt.__metatable = "you can't get and change really metatable"

return Set
```
##### 测试Set
```lua
-- D:\lua_note\chapter20\test_set.lua

package.path = package.path .. ';../lua_note/chapter20/?.lua'
Set = require "set"
local s1 = Set.new({1,2,3,4})
local s2 = Set.new({3,4,5,6})
-- local x = {1,4,6,8,9}
-- for v in pairs(x) do -- 低版本的lua输出结果是content,lua 5.3输出了数字对应的key
--    print(v)
-- end
print(getmetatable(s1))
print(getmetatable(s1) == getmetatable(s2)) -- true
print(s1) -- {1,2,3,4}
print(s2) -- {4,5,6,3} 
-- 可以发现s2乱序了，并不是按我们传入序列的顺序构建的，
-- 原因是我们Set底层实现是key=val形式标记元素存在，存储在table底层的hash表中

s3 = s1 + s2
print(s3) -- {1,2,3,4,5,6}
s4 = s1 - s2
print(s4) -- {4,3}
io.write(tostring(s4)) -- {4,3}
print(Set.sort(s2)) -- {1,2,3,4}
print(getmetatable(s1) == getmetatable(s2)) -- true

print(s4 <= s1) -- true
print(s4 < s1) -- true
print(Set.sort(s2) == s1) -- true

local s3 = s1 + "hello" -- error
print(s3)
local s2 = s1 + 8 -- error
print(s2)
```
#### `__index`

- 如果访问table中没有定义的元素，经常返回nil;
- 原因： 访问字段是通过调用元函数`__index`来完成的，然而一般这个函数没有定义，所以我们访问没有定义的元素得到的是nil;
- 用途： 我们可以定义这个元函数来改变这种行为(比如**访问一个不存在的元素时访问一个默认值**)；
```lua
local tb = {1111,x = 1,g = 10}
local mt = {x = 2,y = 3}
-- 访问一个不存在的元素时返回一个默认值
mt.__index = function ()
    return "hhh"
end
print(tb.haaa,tb.ha,mt.la) -- hhh     hhh     nil


local mt = {}
function new(tb)
    setmetatable(tb,mt)
    return tb
end
local proto = {x = 1,y = 2,width = 3,height = 4}

mt.__index = function (_,key) -- key作为第二个参数，思考：第一个参数是什么？ tip: 这种方式更复杂但更灵活，可以实现多继承；
    return proto[key]
end

local w = new{x = 10,y = 20}
print(w.width) -- 3

local mp = {left = 1024,right = 2048}
mt.__index = mp -- 也可以直接是一个table，这样相当于实现了单继承，继承了mp中的特性
print(w.left) -- 1024
```

- 访问表中元素的顺序: 先访问原始表，如果找到则直接返回，否则访问（调用）元表中的`__index`；
-  `rawget(tb,index)`：即只读原始表，不访问(调用)元表中的`__index`;
```lua
local tb = {x = 1,g = 10}
local mt = {__index = {x = 2,y = 3},z = 4}

setmetatable(tb,mt)
print(tb.x,tb.y,tb.z) -- 1 3, nil,从这里可以看出访问元素的顺序

print(tb.x,tb.y) -- 1       3 --原始表中没有y,去元表中访问（调用）__index
print(rawget(tb,"x"),rawget(tb,"y")) --  1       nil，可见rawget不会访问__index
```
#### `__newindex`

- 当我们给table中一个不存在的 index 赋值时， 如果定义了`__newindex`,则会调用这个元函数而不是直接进行赋值操作,可以借助这个特性保证table只可读;
- 和`__index`类似，如果这个元方法被赋值为另一个table,则不会对原table起作用，而是操作 赋值给这样元方法的table；
```lua
-- read only table
function MakeReadOnly(t)
    mt = {}
    mt.__index = t
    mt.__newindex = function ()
        error("read only")
    end
    return setmetatable(t,mt)
end

local t = {["A"] = 1,["B"] = 2,["C"] = 1024,D = 2048}
MakeReadOnly(t)
t['E'] = "hhhh" --error ： read only

```

- 不执行`__newindex`元方法的赋值操作： `rawset(tb,k,v)`
```lua

local tb = {}
local mt = {
    __newindex = function (tb,k,v)
    rawset(tb,k,v)
    print("use newindex ",tb.b)
    end
}
print("without matetable")
tb.a = 1
print(tb.a)

setmetatable(tb,mt) 
print("with matetable")
tb.b = 2

print("direct rawset")
rawset(tb,"c",3)
print(tb.c)
--[[output:
without matetable
1
with matetable
use newindex    2
direct rawset
3
--]]

```
#### `__call`
```lua
-- __call：当一个table当作函数调用时执行

local tb = {10,40,2,3,5}
local mt = {
    __call = function (t,_)
        local sum = 0
        for _,v in ipairs(t) do
            sum = sum + v;
        end
        print('_ = ',_,", sum = ",sum)
        return sum
    end
}
setmetatable(tb,mt)
tb() -- _ =     nil     , sum =         60
tb("test other") --  _ =     test other      , sum =         60
-- call赋值的函数第一参数就是原来的表，后面可以有其他参数
```
