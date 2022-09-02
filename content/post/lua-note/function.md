---
title: "lua-function"
date: 2021-07-19
description: "lua函数总结"
categories: [
	"Lua",
]	
tags: [
   
]
draft: false
---

### 函数调用形式

一般函数调用都要加括号,但是又两种特殊情况不需要：

- case 1: 传的参数的是 一个 字符串（不是用变量保留的字符串）
- case 2: 传的参数的是 一个 表的构造 `{}`

```lua
print(os.date())

print "hello lua"

function test_case1(str)
    assert(type(str) == "string","args must be a string")
    print(str)
end

 function test_case2(tb)
    assert(type(tb) == "table","args must be a table")
     for k,v in ipairs(tb) do
         print (k,v)
     end
 end
test_case1 "lua ~ "
test_case2 {1,2,3}
```

### 函数特性

- 可以多返回值
- 函数参数个数可以不定的，参数列表 用 "..."

```lua
line = function () -- lambda 函数
    print(string.rep('-',15))
end

line()
-- 函数可以返回多个值
function get_max_min(list)
    min = math.maxinteger
    max = math.mininteger
    for _,v in ipairs(list) do
        max = math.max(max,v) -- max 和 min 函数在低版本的 lua 没有
        min = math.min(min,v)
    end
    return max,min
end


a = {2,5,7,3,9,8,1}
print(get_max_min(a))

-- 函数参数个数可以不定的，参数列表 用 "..."
function my_max(...)
    max = math.mininteger
    for _,v in ipairs(...) do
        max = math.max(max,v)
    end
    return max
end

print(my_max(a))

line()

-- 对于 选择 序列 哪部分内容 可以 用 select
print(select(2,"a","b","c")) --从第 2 个开始选择;output: b c
print(type(select(2,"a","b","c"))) -- string
print(type(select(2,1,2,3))) -- number
x,y = select(2,1,2,3)
print(x,y) -- 2 3
print(select('#',"a","b","c")) -- 若第一个参数为字符 '#'，则返回之后序列的长度，output: 3

-- select 可用于计算不定参数的个数 以及对她进行选择
function select_test(...)
    sum = 0
    for i = 1,select('#',...) do
        sum = sum + select(i,...) -- 此时计算 只会加上所选序列的第一个数，之后的自动忽略
    end
    return sum
end

print(select_test(1,2,3,4,5))

line()
--[[
--注意：select(1,...)返回的是一个序列，但是这里是不可以迭代，思考： ipairs 和 pairs 只能适用于 table ? 
function select_test2(...)
    sum = 0
    for _,v in ipairs(select(1,...)) do  -- error： ipairs(select(1,...)) 不是一个可迭代对象
        sum = sum + v
    end
    return sum
end
print(select_test2(1,2,3,4,5))
--]]
```



