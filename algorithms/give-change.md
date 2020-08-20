---
title: 换零钱的问题
date: 2020-08-15 14:00:00
tags: '算法'
categories:
  - ['算法', '动态规划']
permalink: give-change
photo:
mathjax: true
---

## 简介

通过一系列面值的货币, 计算出某个金额能换成货币的方法

## 分析

对于货币的属性, 可以使用一个所有值都为整数且不重复的数组来表示所有的货币, 例如 $arr=[5, 10, 25]$

- 金额为 aim = 0, 则所有面值货币都不需要, 所以只有 1 种
- 金额为 aim = 3, 则所有组合都不符合, 所以只有 0 种
- 金额为 aim = 15, 则可以 3 个 5 或者 1 个 10, 1 个 5, 所以只有 2 种

<!-- more -->

### 穷举法

通过递归列举所有的值, 就是需要列举数组中的每一项为 0 到全部这过程中, 其余项组成剩余的数的方法数

例如 $arr=[1, 5, 10, 25], aim=1000$

- 用 0 个 1, 让剩下面额组成剩下的 100
- 用 1 个 1, 让剩下面额组成剩下的 999
- 用 2 个 1, 让剩下面额组成剩下的 998
- ...
- 用 1000 个 1, 让剩下面额组成剩下的 0

只要将以上的方法累加起来就是总方法数, 而上述流程剩下的面额符合递归的规则, 这样时间复杂度为 $O(aim^N)$ N 为货币队列的长度

### 剪枝

上述的穷举将所有的可能都进行计算, 在实际分析中, 可以了解到, 如果通过 5 和 10 计算到需要使用 1 和 25 组成剩下的数额时, 经常有重复数据存在, 例如

- 0 个 1 和 2 个 5, 这样需要 10 和 25 来组成剩下的 990
- 5 个 1 和 1 个 5, 这样也需要 10 和 25 来组成剩下的 990

可以知道后续的计算都是一样的, 这样就需要通过额外的记录来避免多余的计算

通过以上规律可以发现, 货币数组是固定的, 总金额是固定, 变化的是当前换算到的货币和剩下的金额, 于是就可以使用这两个变量组成二维数组来保存由剩下的货币换算剩下的金额所有的方法数

这样可以将时间复杂度降低到 $(N \times aim^2)$, N 为货币队列的长度, 额外空间复杂度为 $O(N \times aim)$

### 动态规划

由问题可以知道, 当需要 10 和 25 来组成剩下的 990 时, 所有的方法数是不变的, 也就是在该状态前所有的变化不会影响到后续的变化, 所以该问题的递归状态是无后效性的

可以通过如下规则找到类似该问题的无后效性的动态规划方法

- 通过明确可变参数代表一个递归状态, 这里指通过确定参数即可以确定返回值, 对应该问题的状态, 在固定的队列 arr 和金额 aim 中, 只要确定当前换算的货币和剩下的金额, 这样总会得到明确的结果
- 将上述的参数映射为状态集, 对应该问题就是通过二维数组表示在换了多少种货币, 剩余金额的时候总共有多少种方法
- 找到当前可以明确的最终状态, 对应该问题就是
  - 在剩余金额为 0 时, 也就是不使用任何面额, 所以矩阵第一列的值 $dp[i][0], 0 \leq i \leq N$ 都为 1
  - 仅使用某种货币的情况下, 也就是将剩余的金额换算成某一类货币, 所以需要在符合的金额位置设置为 1, 对应矩阵第一行的值 $dp[0][k \times arr[0]]=1, 0 \leq k \times arr[0] \leq aim, k \in N*$
- 找到执行过程方法, 对应该问题有多个逻辑
  - 不使用 $arr[i]$ 货币, 只使用 $arr[0 \ddots i-1]$ 货币时, 方法数为 $dp[i-1][j]$
  - 使用 1 张 $arr[i]$ 货币, 剩下的使用 $arr[0 \ddots i-1]$ 货币时, 方法数为 $dp[i-1][j-arr[i]]$
  - 使用 2 张 $arr[i]$ 货币, 剩下的使用 $arr[0 \ddots i-1]$ 货币时, 方法数为 $dp[i-1][j-2 \times arr[i]]$
  - 使用 k 张 $arr[i]$ 货币, 剩下的使用 $arr[0 \ddots i-1]$ 货币时, 方法数为 $dp[i-1][j-k \times arr[i]], 0 \leq k \times arr[i] \leq aim, k \in N*$
- 返回当前需要求的状态, 对应该问题知道这个状态就是 $dp[N-1][aim]$

在最差的情况下, 获取 $dp[i][j]$ 需要枚举 $dp[i-1][0 \ddots j]$ 上的所有值所以该解法处理的时间复杂度为 $O(N \times aim^2)$

从 1 到 k 遍历并且累加后得到的值其实就是 $dp[i][j-arr[i]]$ 的值, 所以可以将时间复杂度降低到 $O(N \times aim)$

## 实现

```java
public class Test {
    static int coins(int[] arr, int aim) {
        // 不符合的参数
        if (arr == null || arr.length == 0 || aim < 0) {
            return 0;
        }
        // 缓存数组, 进行记录, 避免重复计算
        int[][] map = new int[arr.length + 1][aim + 1];
        return process(arr, 0, aim, map);
    }

    static int process(int[] arr, int index, int aim, int[][] map) {
        // 方法数
        int res = 0;
        if (index == arr.length) {
            // 在计算当前货币的时候
            // 判断能找出金额的时候返回 1 个方法
            res = aim == 0 ? 1 : 0;
        } else {
            // 记录的值
            int mapValue = 0;
            // 在当前货币可以换的过程中
            for (int i = 0; arr[index] * i <= aim; i ++) {
                // 尝试获取之前的记录值
                mapValue = map[index + 1][aim - arr[index] * i];
                if (mapValue != 0) {
                    // 如果获取的记录为 -1, 则表示该缓存为 0 个方法, 返回
                    // 如果获取的记录不为 -1, 那就表示该缓存的方法, 返回
                    res += mapValue == -1 ? 0 : mapValue;
                } else {
                    // 换下一个货币计算剩下的金额
                    res += process(arr, index + 1, aim - arr[index] * i, map);
                }
            }
        }
        // 将没有方法的设置为 -1 记录
        map[index][aim] = res == 0 ? -1 : res;
        return res;
    }

    static int getCoins(int[] arr, int aim) {
        // 不符合的参数
        if (arr == null || arr.length == 0 || aim < 0) {
            return 0;
        }
        // 设置二维数组
        int[][] dp = new int[arr.length][aim + 1];
        // 当需要换的金额为 0 时, 所有的货币都只有一种方法, 就是不换
        for (int i = 0; i < arr.length; i ++) {
            dp[i][0] = 1;
        }
        // 当只用某种货币换的情况, 将可以换出来的金额对应的位置设置方法 1
        for (int i = 1; arr[0] * i <= aim; i ++) {
            dp[0][arr[0] * i] = 1;
        }
        // 总方法数
        int num = 0;
        for (int i = 1; i < arr.length; i ++) {
            for (int j = 1; j <= aim; j ++) {
                /*num = 0;
                // 累计前一个货币从 1 到全部金额所有的方法数
                for (int k = 0; j - arr[i] * k >= 0; k ++) {
                    num += dp[i - 1][j - arr[i] * k];
                }
                // 第 i 种货币在 j 金额的情况下的总方法数
                dp[i][j] = num;*/
                // 精简上面的步骤
                // 循环累加前一个货币从 1 到全部金额所有的方法数可以简化为 dp[i][j - arr[i]]
                dp[i][j] = dp[i - 1][j];
                dp[i][j] += j - arr[i] >= 0 ? dp[i][j - arr[i]] : 0;
            }
        }
        // 返回所有货币, 换金额的方法数
        return dp[arr.length - 1][aim];
    }

    public static void main(String[] args) {
        int[] arr = {1, 5, 10, 25};
        System.out.println(coins(arr, 1000));
        System.out.println(getCoins(arr, 1000));
    }
}
```

## 结果

```sh
142511
142511

Process finished with exit code 0
```
