---
title: "洗牌和全排列的联系"
date: 2022-09-21T23:36:00+08:00
description: "之前写到洗牌算法(Fisher-Yates洗牌)和全排列算法，突然发现它们交换过程惊奇一致。"
categories: [
	"algorithm"
]	
tags: [
   "shuffle","permutation"
]
draft: true
---
## 洗牌算法

> 假设有`n`个元素的序列，要对它们`shuffle`

+ Fisher-Yates洗牌基本思路： 从下标`[curIndex, n)`随机选一个使其对应的数字和当前下标`curIndex`的数字进行交换
+ 代码实现：

```cpp
vector<int> shuffle(vector<int>& nums) {
    int n = nums.size();
    for(int i = 0; i < n; ++i) 
    {
        int randIndex = i + rand() % (n - i); // 在[curIndex, n)之间
        std::swap(nums[i], nums[randIndex]);
    }
    return nums;
 }
```
+ 每个元素都等概率出现在任一位置（或者说：**每一种排列**都等概率的出现）证明：

  因为显然有：任一元素在第1个位置的概率为:
  $$
  \frac{1}{n}
  $$
  

  

  任一元素第2个位置的概率为：
  $$
  \frac{n-1}{n} * \frac{1}{n - 1} = \frac{1}{n}
  $$
  

  任一元素第3个位置的概率为：
  $$
  \frac{n-1}{n} * \frac{n-2}{n - 1} * \frac{1}{n - 2} = \frac{1}{n}
  $$
  ...

  所以每个元素都等概率出现在任一位置。

## 全排序算法

> 假设有`n`个元素的序列， 返回它的所有可能排序，可以按照任意顺序

+ 基本思想： 我们看成**填一个长度为n的序列， 从左往右每一个位置每次选一个没有使用过的元素填入**

+ 版本1：

  + 以前我经常通过借助一个`bool`数组来标记元素是否使用, 具体实现如下：

      ```cpp
      class Solution {
      public:
          vector<vector<int>> permute(vector<int>& nums) {
              int n = nums.size();
              vector<bool> vis(n, false);
              vector<vector<int> > ret;
              vector<int> item;
              function<void(int)> dfs = [&](int curCount)
              {
                  if(curCount == n)
                  {
                      ret.push_back(item);
                      return;
                  }
                  for(int i = 0; i < n; ++i)
                  {
                      if(vis[i]) continue;
                      vis[i] = true;
                      item.push_back(nums[i]);
                      dfs(curCount + 1);
                      item.pop_back();
                      vis[i] = false;
                  }
              };
              dfs(0);
              return ret;
          }
      };
      ```

+ 版本2： 
  + 现在我一般会这么实现（一方面空间复杂度更优，另一方面写起来也更简洁...）：

    ```cpp
    class Solution {
    public:
        vector<vector<int>> permute(vector<int>& nums) {
            int n = nums.size();
            vector<vector<int> > ret;
            function<void(int)> dfs = [&](int curIndex)
            {
                if(curIndex == n-1) // 这里写n也是可以，但是没有必要，因为它后面没有其他元素可swap
                {
                    ret.push_back(nums);
                    return;
                }
                for(int i = curIndex; i < n; ++i) // [curIndex, n), 和前面提到的shuffle一致！！！
                {
                    std::swap(nums[curIndex], nums[i]);
                    dfs(curIndex + 1);
                    std::swap(nums[curIndex], nums[i]);
                }
            };
            dfs(0);
            return ret;
        }
    };
    ```
    
  + **版本二代码 关键过程 现在我是 通过 Fisher-Yates洗牌思路**来理解记忆。
  
  + 当然，版本二也可以这样理解，将序列看成两部分：
  
    + 一部分已经确定下来了：前面部分`[0, curIndex)`已经确定了
    + 另一部分没有确定下来： 那就对后面部分`[curIndex, n)`全排列， 可通过每一个元素和`curIndex`处元素交换来实现，递归返回时交换回来。

## 总结

+ 洗牌本身要求就是**每一种排列**都等概率的出现，而全排列就是把**每一种排列**枚举出来。
+ 如果你发现了前面提到的Fisher-Yates洗牌和版本2的全排列的相同之处，相信以后你只要会**两者中之一**， 那么两者大概率都能够实现出来。
+ 对于全排列算法，涉及到全排列的去重，字典序，非递归写法（字典序求解下一个排列），我以前有个简单总结：[Algorithm of permutation(全排列算法)](http://t.csdn.cn/Y7ixU)。

