---
title: "lua-table"
date: 2021-07-19
description: "lua table总结"
categories: [
	"Lua",
]	
tags: [
   
]
draft: false
---

### table的key以及存储机制
-  `{}`表示table,它可以以除nil外的任何对象作为key；    table的底层由 数组 和 hash表 构成；
- 当增加一个很大的key使得存储空间利用率低于50%时，这个key会被hash，然后放入hash表存储；
- 0，负数，string 作为key时都是被放在hash表存储；
- table的底层的 数组 或者 hash表 满时，都是以2倍扩容；
- hash表大小是2的倍数，只有当hash表满了，才会重新分配 数组和 hash表的空间；

    ref:[https://blog.csdn.net/zr339361504/article/details/52432163](https://blog.csdn.net/zr339361504/article/details/52432163)
### table计算长度: 
> 采用#table_name 计算table长度

注意：

1. 对于带nil的列表，用#计算其长度是不可靠的,
所以它只适用于没有nil的列表;
1. 没有正整数作为key的列表长度为0；
```lua
b = {1,5,nil,10,nil,1024,nil,nil}
print(#b) -- 6 
c = {1,5,nil,10,nil,1024,nil,nil,x = 3}
print(#c) -- 6
-- 以上两种 发现是返回最后不是nil处的index

d = {1,5,nil,10,nil,1024,nil,nil,x = 3,233} -- 等价于计算{1,5,nil,10,nil,1024,nil,nil,233}
print(#d) -- 9
-- 发现键值对的并没有算进去，因为它存在底层的hash表中，#此时只计算底层的array的长度

e = {x = 1,y = 2,['z'] = 3}
print(#e) -- 0
f = {x = 1,y = 2,[1] = 2}
print(#f) --1
g = {x = 2,[2.1] = 3,[-1] = 3}
print(#g) --0,发现并没有计算hash表部分
```
### #求长度伪代码：
```lua
if 数组最后一位 == nil then
    则二分查找往前找到一个不是nil的返回长度
else if 数组最后一位 ~=nil and 散列桶部分 == nil then
    return 数组长度
else
    计算散列桶部分的长度
local function 计算散列桶部分的长度()
    从数组长度+1 开始查找, 同样二分查找  （只针对散列桶部分的key为正整数的数据）
end
```
所以使用table时：
1. 尽量放同一类元素，不要 使得 底层的array 和hash表 都被使用;
2. 存在nil的table 求长度不稳定;
3. 尽量使用数组部分，尽量避免重新散列操作（hash表满后 会 重新分配底层的array和hash表的内存，严重影响效率）;

### table 的遍历方式

- 方式1:
```lua
print('-----normal--------')
-- 与ipairs 等价： 遇到nil终止无法遍历到 键值对元素
for i = 1 , #tb do
    print(i , tb[i])
end
```

- 方式2：
```lua
-- ipairs 遇到nil终止，无法遍历到 键值对元素
print("-----ipairs--------")
for k,v in ipairs(tb) do
    print(k,v)
end
```

- 方式3：
```lua
-- k,v形式遍历table,能遍历到所有元素 
print("-----paris--------")
for k,v in pairs(tb) do
    print(k,v)
end
```
### 测试代码
```lua
tb = {}
tb['x'] = 1
tb[2.1] = 2021
print(tb[2.1],tb.x)
tb[2] = 2
print(tb[2])

-- table是匿名对象，保存table的变量是对表的引用，当表的引用计数为0时会被gc自动回收
tb2 = tb
tb = nil
print(tb2['x']) -- 依然有效
tb2.y = 10 -- 等价于tb2['y'] = 10
print(tb2.y,tb2['y'])

-- table 构造列表可以是 单个值 以及 键值对 的形式
list = {x = 1,"hello",y = 2,math.pi}
print(list.x)

-- 注意: 构造列表，起始索引1 从 第一个 非键值对的元素 算起,依次对这类元素进行index++
print(list[0],list[1]) -- nil     hello
print(list[2],list[-1])  -- 3.1415926535898 nil

list[-1] = 1024
list[0] = 2021
list[1] = "world"

-- k,v形式遍历table,能遍历到所有元素 
print("-----paris--------")
for k,v in pairs(list) do
    print(k,v)
end
-- ipairs 遇到nil终止，无法遍历到 键值对元素
print("-----ipairs--------")
for k,v in ipairs(list) do
    print(k,v)
end

--[[ output:
    -----paris--------
1       world
2       3.1415926535898
y       2
x       1
-1      1024
0       2021
-----ipairs--------
1       world
2       3.1415926535898
--]]

print('-----normal--------')
-- ipairs 等价于以下我们常用的方式:
for i = 1 , #list do
    print(i , list[i])
end


```
### 常用函数
```lua
--useful functions in table library

-- 增加 和 移除
t = {1,2,3,4,5,6}
print(t[6])
table.insert(t,6,7) -- table, pos ，val
print(t[6],t[7])
x = table.remove(t,7)  -- table，pos,返回该位置的值
print(x)
table.insert(t,8) -- table,val ：默认插在最后
print(t[#t])
--[[
    t = {}
    for line in io.lines() do -- ctrl+z 结束输入
        table.insert(t,line)
    end
    
    print(#t)
    for k,v in pairs(t) do
        print(k,v)
    end
--]]

print(string.rep('-',10))

function print_table(a)
    for k,v in ipairs(a) do
        print(k,v)
    end
    
end
-- 移动元素
a = {1,2,3,4,5,6}
-- move在5.3版本有，但是之前的5.1版本却不存在
print(#a)
table.move(a,1,#a,2) -- a[1...#a] = a[2...2+#a],即整体后移一位
print(#a)
print_table(a) -- 可以发现空出的位置a[1]的值没有发生变化

print(string.rep('-',10))

a[1] = 1024  --这时相当于 头部 增加一个元素

table.move(a,2,#a,1)
print_table(a) -- 空出的位置a[#a]值 依然不变,a[1]位置上的值被覆盖了
print(#a)
a[#a] = nil -- 尾部置nil,此时 真正意义上 删掉了一个元素 而且 是 头部元素
print(#a)


-- 由上面两个测试 可以大致推断table中move 实现
function move(a,s,e,pos) 
    if pos == s then
        return a
    end
    while s <= e do
        a[pos] = a[s]
        pos = pos + 1
        s = s + 1
    end
    return a
end

print(string.rep('-',10))

a = {1,2,3,4,5,6}
-- print(table.move(a,3,5,2) == move(a,3,5,2))  -- true
-- print_table(a) -- after: {1,3,4,5,5,6}

print(table.move(a,1,#a,2) == move(a,1,#a,2))  -- true
print_table(a)

print(string.rep('-',10))
b = {2,6,7,3,9,2,1}
-- 排序 : 默认升序
table.sort(b)
print_table(b)
print(string.rep('-',10))

-- 自己写比较规则
table.sort(b,function (a,b)
    return a > b -- 降序
end)
print_table(b)

-- 将元素连接成一个字符串
print(table.concat(b,'-',1,#b))
```

