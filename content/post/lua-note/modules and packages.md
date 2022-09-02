---
title: "lua-modules and packages"
date: 2021-07-23
description: "lua模块和包总结"
categories: [
	"Lua",
]	
tags: [
   
]
draft: false
---

> modules and packages , lua 5.1 后 提供了一系列模块和包策略

-  `package` 是  `modules` 的集合
### 加载使用模块
```lua
local m = require "math"
print(m.sin(3.14)) -- 0.0015926529164868
```
### require 的参数和返回值
#### 参数

- `require`只需要传入一个参数
```lua
math = require("math") -- 只有当这个参数是纯字符串时，括号才是可选的
string = require "string"
-- 解释器是通过式预加载的方式载入模块，所以我们可以修改预加载好的模块标识而不影响原来的库
math.sin = print
math.sin(1,2,3) -- 1       2       3

cos = require "math".cos -- 也可以加载特定的函数，这种参数不能加()
print(cos(3.14)) -- -0.99999873172754
```
#### 返回值

-  如果加载成功，模块有返回值，就是模块返回值的类型，如果没有返回值，返回值为`boolean`类型;
- 模块加载失败会报错 ;
- [ref]([https://blog.csdn.net/qweewqpkn/article/details/49050507](https://blog.csdn.net/qweewqpkn/article/details/49050507));
#### 其他

- 一旦某个模块已经加载成功，其他需要用这个模块的调用都只需返回这个模块，而不需执行其他代码（比如查找模块等）
- 加载操作时查找lua模块的路径存在`package.path`中,加入 查找路径 示例：
```lua
package.path = package.path .. ';D:/lua_note/chapter17/?.lua' -- 注意添加路径前需要加";"
```

-  如果在lua文件中没有找到这个模块，会去搜索C的模块，搜索路径为：`package.cpath`
- 为了避免相同模块被加载两次，可以这样**设置**：
```lua
package.loaded.modname = nil
```
### 编写lua模块
#### 简单示例
```lua
 -- D:/lua_note/chapter17/complex.lua:一个简单的复数模块

 local M = {}
 --package.loaded[...] = M -- 前面放进去后面就不用返回值

 -- 创建一个新的复数:z = ar+bi -- 实部：[r] = r,虚部：[i] = i
 local function new(r,i)
     return {r=r,i=i}
 end

 M.new = new --将new函数加入模块

-- 纯虚数 z = i
 M.i = new(0,1)

-- 复数相加
 function M.add(c1,c2)
     return new(c1.r+c2.r,c1.i+c2.i)
 end

 -- 复数相减
 function M.sub(c1,c2)
    return new(c1.r-c2.r,c1.i-c2.i)
end

-- 复数相乘
function M.mul(c1,c2)
    return new(c1.r*c2.r-c1.i*c2.i,c1.r*c2.r+c1.i*c2.i)
end

local function inv(c)
    local n = c.r^2 + c.i^2
    return new(c.r/n,-c.i/n)
end

-- 复数相除
function M.div(c1,c2)
   return M.mul(c1,inv(c2))
end

function M.tostring(c)
    return string.format("(%g,%g)",c.r,c.i) -- %g(%G) : 接受一个数字并将其转化为%e(%E,对应%G)及%f中较短的一种格式
end

return M -- 加载时返回模块
```
#### 使用上面编写的模块
```lua
-- D:/lua_note/chapter17/use_complex.lua
package.path = package.path .. ';D:/lua_note/chapter17/?.lua'
comp = require "complex"
print(comp.tostring(comp.add(comp.new(1,2),comp.i))) -- (1,3)
```
