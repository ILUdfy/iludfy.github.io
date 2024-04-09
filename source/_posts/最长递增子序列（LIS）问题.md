---
title: 最长递增子序列（LIS）问题
date: 2024-04-06 13:36:40
tags:
 - LIS
 - 二分查找
categories: 算法
---
[leetcode 300 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/description/)

# 题目描述

> 给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
>
> 子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。

这个题目第一反应是动态规划，dp数组表示以nums数组第i个元素结尾的子序列，最长递增长度为多少。
状态转移方程为：
$$
dp[i] = \max_{nums[j]<nums[i]} dp[j]+1
$$
时间复杂度为 $$o(n^2)$$，空间复杂度为$$o(n)$$

# 贪心+二分查找

二分查找可以达到$$O(nlogn)$$的复杂度，思路如下：

1. 维护一个辅助数组 **p** ，它的每一项 $$p[i]$$ 的含义是，所有长度为 **i+1 **的上升子序列的末尾元素中的最小值；
2. 遍历数组$$num$$ , 如果 $$nums[i]$$ 大于数组中最后一个元素,那么直接把该元素放到数组尾部 ; 否则使用二分查找寻找 $$nums[i]$$ 插入数组 ***p*** 的位置 ***j***  , 并使$$p[j]=nums[i]$$

可知 , 数组 **p** 是严格递增的 . 

这种算法的思想是这样的:

​	在寻找最长递增子序列的过程中 , 我们希望子序列的增长尽可能的慢 , 这样在后续的遍历中才能放下更多的元素 。因此我们在遍历时，会使用$$nums[i]$$来替换$$p[j]$$的值，目的是使子序列 ***p*** 上升得尽可能慢，例如：

​	我们在遍历时已经得到了数组 ***p*** 为：
$$
p = [1,3,5,7,9]
$$
​	现在遍历到$$nums[i]=8$$,显然使用二分查找后，找到元素 8 应该在 ***p*** 中的下标为 4， 那么我们用 8 来替换 $$p[4]$$，得到：
$$
p = [1,3,5,7,8]
$$
​	这样就保证了在不降低递增子序列长度的前提下，使 ***p*** 增长得尽可能慢。

​	**注意：最后得到的数组 p 并不是最后的递增子序列，只能保证它的长度等于最大递增子序列的长度！**

​	考虑下面的数组：
$$
nums = [1,3,5,7,9,4]
$$
​	我们在遍历到下标 4 的位置时，得到的数组 ***p*** 为
$$
p = [1,3,5,7,9]
$$
​	继续遍历到$$nums[5]=4$$时，按照算法，应该把它插入到$$p[2]$$的位置，于是得到数组 ***p*** 为
$$
 p = [1,3,4,7,9]
$$
​	这显然不是最后得到的最大递增子序列，只是它的长度是在遍历过程中出现过的最长递增子序列的长度而已。

​	

实现的代码如下：

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        List<Integer> list = new ArrayList<>();
        for(int i=0;i<nums.length;i++){
            if(list.size()==0||nums[i]>list.get(list.size()-1)){
                list.add(nums[i]);
            }else{
                list.set(sort(list, nums[i]), nums[i]);
            }
        }
        return list.size();
    }

    public int sort(List<Integer> list, int target){
        int left=0,right=list.size(),mid;
        while(left<right){
            mid = left + ((right-left)>>1);
            if(list.get(mid)<target){
                left = mid + 1;
            }else{
                right = mid;
            }
        }
        return left;
    }
}
```



