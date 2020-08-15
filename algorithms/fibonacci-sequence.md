---
title: 斐波那契数列
date: 2020-08-09 10:00:00
tags: '算法'
categories:
  - ['算法', '动态规划']
permalink: fibonacci-sequence
photo:
mathjax: true
---

## 简介

斐波那契数列 (Fibonacci sequence) , 又称黄金分割数列, 因数学家莱昂纳多·斐波那契 (Leonardoda Fibonacci) 以兔子繁殖为例子而引入, 故又称为 "兔子数列" , 指的是这样一个数列: $0, 1, 1, 2, 3, 5, 8, 13, 21, 34, \dots$ 在数学上, 斐波那契数列以如下被以递推的方法定义: $F(1)=1, F(2)=1, F(n)=F(n - 1)+F(n - 2) (n \geq 3, n \in N \ast)$

<!-- more -->

## 分析

### 递归解

使用递归的方法定义, 可以很轻松的得到结果, 时间复杂度为 $O(2^N)$

### 遍历解

可以通过正序求解的方式, 使用两个指针一直指向倒数第二, 第三项, 这样时间复杂度为 $O(N)$

### 矩阵解

由定义 $F_n=F_{n-1}+F_{n-2}$ 是一个二阶递推数列, 可以使用矩阵乘法的形式表示, 且状态矩阵为 $2 \times 2$ 的矩阵

$$
\begin{cases}
  F_n = 0 & n=0 \\
  F_n = 1 & 1 \leq n \leq 2 \\
  F_n = F_{n-1} + F{n-2} & n \geq 3
\end{cases}
$$

可得当 $n \geq 3$ 时

$$
\left[
  \begin{array}{c}
    F_3 \\
    F_2
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    a&b \\
    c&d
  \end{array}
\right]
\times
\left[
  \begin{array}{c}
    F_2 \\
    F_1
  \end{array}
\right]
$$

$$
\left[
  \begin{array}{c}
    F_4 \\
    F_3
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    a&b \\
    c&d
  \end{array}
\right]
\times
\left[
  \begin{array}{c}
    F_3 \\
    F_2
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    a&b \\
    c&d
  \end{array}
\right]^2
\times
\left[
  \begin{array}{c}
    F_2 \\
    F_1
  \end{array}
\right]
$$

将 $F_1=1, F_2=1, F_3=2, F_4=3$ 代入

$$
\left[
  \begin{array}{c}
    3 \\
    2
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    a&b \\
    c&d
  \end{array}
\right]
\times
\left[
  \begin{array}{c}
    2 \\
    1
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    a&b \\
    c&d
  \end{array}
\right]^2
\times
\left[
  \begin{array}{c}
    1 \\
    1
  \end{array}
\right]
$$

可以求出状态矩阵

$$
\left[
  \begin{array}{cc}
    a&b \\
    c&d
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    1&1 \\
    1&0
  \end{array}
\right]
$$

依次乘下去, 可以得到一般形式

$$
\left[
  \begin{array}{c}
    F_n \\
    F_{n-1}
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    1&1 \\
    1&0
  \end{array}
\right]^{n-2}
\times
\left[
  \begin{array}{c}
    F_2 \\
    F_1
  \end{array}
\right]
=
\left[
  \begin{array}{cc}
    1&1 \\
    1&0
  \end{array}
\right]^{n-2}
\times
\left[
  \begin{array}{c}
    1 \\
    1
  \end{array}
\right]
$$

所以可以将求斐波那契数列第 N 项的问题转换为求一个矩阵 N 次方的问题, 时间复杂度为 $O(logN)$

#### 整数 N 次方例子

求矩阵 N 次方的问题与求整数 N 次方一样, 只是计算细节不一样, 例如在时间复杂度为 $O(logN)$ 的情况下计算整数的 N 次方

计算 $10^75$ 的规则如下

1. 75 的二进制数字式是 1001011
2. $10^75 = 10^64 \times 10^8 \times 10^2 \times 10^1$

根据过程, 先计算 $10^1$, 再根据 $10^1$ 计算出 $10^2$, 再根据 $10^2$ 计算出 $10^4$, 依次计算 $10^8, 10^16, 10^32, 10^64$

需要计算的次数是 75 的二进制形式的位数

然后再将 75 的二进制数中每一位上是 1 的结果进行累乘

对于矩阵的算法同理

在程序中可以通过位移计算来获得每一位二进制数字

例如

```java
// 00000000000000000000000000000101
int a = 5;
// 与操作 0001
// 结果 1
a & 1;
// 向右移动 1 位
// 00000000000000000000000000000010
a >>= 1;
// 与操作 0001
// 结果 0
a & 1;
```

矩阵乘法方法是设 A 为 $m \times p$ 的矩阵，B 为 $p \times n$ 的矩阵，那么称 $m \times n$ 的矩阵 C 为矩阵 A 与 B 的乘积，记作 $C=AB$，其中矩阵 C 中的第 i 行第 j 列元素可以表示为

$$
(AB)_{ij} = \sum_{k=1}^p a_{ik}b_{kj} = a_{i1}b_{1j} +  a_{i2}b_{2j} + \cdots + a_{ip}b_{pj}
$$

用程序的方式表示, 就是采用三个循环分别遍历 i, j, k, 然后不断的进行当前项的累加

实现如下

```java
int[][] matrixPower(int[][] m, int p) {
    int[][] res = new int[m.length][m[0].length];
    // 填充单位矩阵, 即对角线为 1, 等于 1
    for (int i = 0; i < res.length; i ++) {
        res[i][i] = 1;
    }
    int [][] tmp = m;
    // 将 p 次方的二进制数进行向右位移一位操作, 这样会将二进制数每位都循环放到最右位
    for (; p != 0; p >>= 1) {
        // 与 1 进行与操作, 判断上面的最右位是否为 1, 1 的话条件成立
        if ((p & 1) != 0) {
            // 只要是 p 的二进制位为 1 的, 进行矩阵乘法
            res = muliMatrix(res, tmp);
        }
        // 平方
        tmp = muliMatrix(tmp, tmp);
    }
    return res;
}

int[][] muliMatrix(int[][] m, int[][] m2) {
    // 两个矩阵相乘
    int[][] res = new int[m.length][m2[0].length];
    for (int i = 0; i < m.length; i ++) {
        for (int j = 0; j < m2[0].length; j ++) {
            for (int k = 0; k < m2.length; k ++) {
                res[i][j] += m[i][k] * m2[k][j];
            }
        }
    }
    return res;
}
```

## 实现

```java
public class Fibonacci {
    static int fibonacci(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 3) {
            return 1;
        }
        // 递归
        return fibonacci(n - 1) + fibonacci(n - 2);
    }

    static int fibonacci2(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 3) {
            return 1;
        }
        int tmp = 0;
        int pre = 1;
        int res = 1;
        // 遍历
        for (int i = 3; i <= n; i ++) {
            tmp = res;
            res = res + pre;
            pre = tmp;
        }
        return res;
    }

    static int[][] matrixPower(int[][] m, int p) {
        int[][] res = new int[m.length][m[0].length];
        // 填充单位矩阵, 即对角线为 1, 等于 1
        for (int i = 0; i < res.length; i ++) {
            res[i][i] = 1;
        }
        int [][] tmp = m;
        // 将 p 次方的二进制数进行向右位移一位操作, 这样会将二进制数每位都循环放到最右位
        for (; p != 0; p >>= 1) {
            // 与 1 进行与操作, 判断上面的最右位是否为 1, 1 的话条件成立
            if ((p & 1) != 0) {
                // 只要是 p 的二进制位为 1 的, 进行矩阵乘法
                res = muliMatrix(res, tmp);
            }
            // 平方
            tmp = muliMatrix(tmp, tmp);
        }
        return res;
    }

    static int[][] muliMatrix(int[][] m, int[][] m2) {
        // 两个矩阵相乘
        int[][] res = new int[m.length][m2[0].length];
        for (int i = 0; i < m.length; i ++) {
            for (int j = 0; j < m2[0].length; j ++) {
                for (int k = 0; k < m2.length; k ++) {
                    res[i][j] += m[i][k] * m2[k][j];
                }
            }
        }
        return res;
    }

    static int fibonacci3(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 3) {
            return 1;
        }
        int[][] base = {{1, 1}, {1, 0}};
        int[][] res = matrixPower(base, n - 2);
        return res[0][0] + res[1][0];
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i ++) {
            System.out.println(fibonacci(i));
        }
        System.out.println("---");
        for (int i = 0; i < 10; i ++) {
            System.out.println(fibonacci2(i));
        }
        System.out.println("---");
        for (int i = 0; i < 10; i ++) {
            System.out.println(fibonacci3(i));
        }
    }
}
```

## 结果

```sh
0
1
1
2
3
5
8
13
21
34
---
0
1
1
2
3
5
8
13
21
34
---
0
1
1
2
3
5
8
13
21
34

Process finished with exit code 0
```
