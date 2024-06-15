---
title: 一道缝合怪算法题的总结-LeetCode2818
date: 2024-04-07 19:40:11
tags:
 - 素数
 - 单调栈
 - 快速幂
 - 贪心
categories: 算法
mathjax: true
---

[2818.操作使得分最大]([2818. 操作使得分最大 - 力扣（LeetCode）](https://leetcode.cn/problems/apply-operations-to-maximize-score/description/))

## 题目描述

给你一个长度为 `n` 的正整数数组 `nums` 和一个整数 `k` 。

一开始，你的分数为 `1` 。你可以进行以下操作至多 `k` 次，目标是使你的分数最大：

- 选择一个之前没有选过的 **非空** 子数组 `nums[l, ..., r]` 。
- 从 `nums[l, ..., r]` 里面选择一个 **质数分数** 最高的元素 `x` 。如果多个元素质数分数相同且最高，选择下标最小的一个。
- 将你的分数乘以 `x` 。

`nums[l, ..., r]` 表示 `nums` 中起始下标为 `l` ，结束下标为 `r` 的子数组，两个端点都包含。

一个整数的 **质数分数** 等于 `x` 不同质因子的数目。比方说， `300` 的质数分数为 `3` ，因为 `300 = 2 * 2 * 3 * 5 * 5` 。

请你返回进行至多 `k` 次操作后，可以得到的 **最大分数** 。

由于答案可能很大，请你将结果对 `109 + 7` 取余后返回。

**提示：**

- `1 <= nums.length == n <= 10^5`
- `1 <= nums[i] <= 10^5`
- `1 <= k <= min(n * (n + 1) / 2, 10^9)`

## 问题分解

1. 题目最终目的是使得分最大，而得分是通过乘以数组元素实现的，也就是要尽可能乘以更大的元素，考虑维护一个大根堆
2. 一个元素可以被选择多少次由两个指标决定：一是和左右两边质数分数的大小有关，这决定了选中该元素的子数组最多可以向两侧扩张多少，考虑维护一个单调栈；二是和非空子数组之前是否被选择过有关，涉及到了数学的计数原理
3. 质数分数的大小计算可以先维护一个质数表

## 质数表的预处理



​	判断一个数是否为质数，通常采用的是从2遍历到根号n，这样做的时间复杂度为$$O(\sqrt{n})$$，还可以通过提前维护素数表来实现：

>  题目中数组元素的最大值为100000，可以提前创建一个boolean型数组`boolean[] isPrime = new boolean[100001];`，并将所有元素置为`true`，按照以下方法处理：
>
> 1. 从2开始向100000/2遍历，每次遍历到`isPrime[i]`时，将`i`的倍数`isPrime[i*j] ,j=2,...n`置为`false`表示`i*j`是合数
> 2. 若当遍历到`isPrime[i]`时，`isPrime[i]`已经为`false`，代表`i`是合数 , 且它的倍数一定在之前已经被遍历过 , 所以直接跳过即可

经过这一步骤，数组`isPrime`已经可以判断数`i`是否为质数



但本题不仅要判断是否为质数 , 还需要求该数的所有质因数的数量，所以对预处理方法稍作修改：

> 创建一个int型数组`int[] scores = new int[100001]`，存放每个数的质数分数，基于上述方法做如下修改：
>
> 1. 遍历到`scores[i]`时，将`i`的倍数`scores[i*j] ,j=1，2,...n`加一
> 2. 遍历到`scores[i]`时，如果`scores[i]==0`代表之前从来没有被访问过，也就是`i`是素数，但考虑到素数本身也是它的质数分数，所以要将`scores[i]++`，这也就是为什么这里`j=1，2,...n`而上面的布尔型数组的`j=2,...n`

经过这一步骤，数组`scores`已经存放各个数字的质数分数

## 单调栈

​	由于要判断数组元素左右两侧距离它最近的质数分数大于等于它的数（由于题目要求如果多个元素质数分数相同且最高，选择下标最小的一个，所以左侧是大于等于，右侧是大于），所以需要维护两个数组`int[] left,right`，这两个数组的初始化由单调栈完成

​	我最开始的思路是：

1. **从左往右 **遍历数组`nums`，按照质数分数 **递减** 维护一个单调栈，栈中存数组下标，`left[i]`的值等于栈中第一个质量分数大于等于`nums[i]`的下标
2. 再 **从右往左** 遍历数组`nums`，按照按照质数分数 **递增** 维护一个单调栈，`right[i]`的值等于栈中第一个质量分数大于等于`nums[i]`的下标

​	代码如下：

```java
		int[][] scores = new int[n][2];//这里scores第一列是left数组，第二列是right数组
        Stack<Integer> stack = new Stack<>();
        for(int i=0;i<n;i++){
            while(!stack.isEmpty()&&isPrime[nums.get(stack.peek())]<isPrime[nums.get(i)]){
                stack.pop();
            }
            scores[i][0] = stack.isEmpty()?-1:stack.peek();
            stack.add(i);
        }
        stack.clear();
        for(int i=n-1;i>=0;i--){
            while(!stack.isEmpty()&&isPrime[nums.get(stack.peek())]<=isPrime[nums.get(i)]){
                stack.pop();
            }
            scores[i][1] = stack.isEmpty()?n-1:isPrime[nums.get(stack.peek())]==isPrime[nums.get(i)]stack.peek():stack.peek()-1;
            stack.add(i);
        }
```



但是看了灵神的题解发现一次遍历就可以做到（灵神是真的强），思路是这样的：

1. `right`数组所有值初始化为`nums`长度
2. 栈中放一个初始元素`-1`，方便数组第一个元素
3. **从左往右** 遍历数组`nums`， 由于栈中存放的是数组下标，因此在因为栈顶元素的质数分数小于当前元素`nums[i`]时，可以直接把弹出的数组下标的`right[stack.pop()] = i`位置幅值当前遍历的下标，当前位置即是栈中元素右侧的第一个更大的元素
4. 栈顶质量分数小的下标弹出后，栈顶元素即是当前元素`nums[i]`左侧第一个质数分数大于等于当前元素的下标，令`left[i]=stack.peek()`即可
5. 把下标`i`放入栈中

代码如下：

```java
		int[][] scores = new int[n][2];
        for(int i=0;i<n;i++){
            scores[i][1] = n;
        }		
		Stack<Integer> stack = new Stack<>();
        stack.push(-1);
        for(int i=0;i<n;i++){
            while(stack.size()>1&&isPrime[nums.get(stack.peek())]<isPrime[nums.get(i)]){
                scores[stack.pop()][1]=i;
            }
            scores[i][0] = stack.peek();
            stack.push(i);
        }
```

## 快速幂

快速幂算法比较经典了，直接贴代码：

```java
public long fastPow(long x,int y){
        long ans = 1;
        while(y>0){
            if((y&1)==1){
                ans = (x*ans)%mod;
            }
            x = (x*x)%mod;
            y >>= 1;
        }
        return ans;
    }
```

## 其他



大根堆没什么好说的，创建一个优先队列指定，按照质数分数逆序存数组下标就行

不重复的子数组通过计数原理实现

对于每个元素`nums[i]`，只要子数组的下界在`(left[i],i]`之间，上界在`[i,right[i])`之间，那么这个子数组最后选择的元素一定是`nums[i]`，一共有`(i-left[i])*(right[i]-i)`个这样的子数组；并且对于整个数组`nums`，选中每个元素的子数组集合之间一定没有交集（因为每个子数组只会选中一个元素），所以可以确定子数组之前一定是没有选择过的。

那么在`k`不为0时，从大根堆里弹出下标`i`，对于`nums[i]`，它对结果产生的贡献就是乘以`pow(nums[i], Math.min(k,(i-left[i])*(right[i]-i)))`，循环直到`k`为0即可。

## 代码

最终整道题的代码如下：

用时击败28.57%的java用户，不高，但时间复杂度依然是`O(nlogn)`：

```java
class Solution {
    int mod = 1000000007;

    static int[] isPrime = new int[100001];

    static{
        for(int i=2;i<isPrime.length;i++){
            if(isPrime[i]==0){
                for(int j=1;j*i<isPrime.length;j++){
                    isPrime[i*j]++;
                }
            }
        }
    }

    public int maximumScore(List<Integer> nums, int k) {
        int n = nums.size();
        Queue<Integer> queue = new PriorityQueue<>(new Comparator<Integer>(){
            @Override
            public int compare(Integer o1, Integer o2){
                return nums.get(o2)-nums.get(o1);
            }
        });
        int[][] scores = new int[n][2];
        for(int i=0;i<n;i++){
            scores[i][1] = n;
            queue.offer(i);
        }
        
        Stack<Integer> stack = new Stack<>();
        stack.push(-1);
        for(int i=0;i<n;i++){
            while(stack.size()>1&&isPrime[nums.get(stack.peek())]<isPrime[nums.get(i)]){
                scores[stack.pop()][1]=i;
            }
            scores[i][0] = stack.peek();
            stack.push(i);
        }
        // System.out.println(Arrays.deepToString(scores));
        int curMax = 0 ,curMaxIndex = 0, subCount = 0;
        long ans = 1;
        while(k>0){
            curMaxIndex = queue.poll();
            curMax = nums.get(curMaxIndex);
            subCount = (curMaxIndex - scores[curMaxIndex][0])*(scores[curMaxIndex][1] - curMaxIndex);
            if(subCount>k){
                ans = (ans*fastPow(curMax, k))%mod;
                // System.out.println("乘以"+curMax+"的"+k+"次方");
                k = 0;
            }else{
                ans = (ans*fastPow(curMax, subCount))%mod;
                // System.out.println("乘以"+curMax+"的"+subCount+"次方");
                k -= subCount;
            }
        }
        return (int)ans;
    }

    public long fastPow(long x,int y){
        long ans = 1;
        while(y>0){
            if((y&1)==1){
                ans = (x*ans)%mod;
            }
            x = (x*x)%mod;
            y >>= 1;
        }
        return ans;
    }
}
```

