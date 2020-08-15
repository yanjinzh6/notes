---
title: 类斐波那契数列问题
date: 2020-08-09 14:00:00
tags: '算法'
categories:
  - ['算法', '动态规划']
permalink: similar-fibonacci-sequence
photo:
mathjax: true
---

## 简介

斐波那契数列比较了解, 都知道就是求 $F(1)=1, F(2)=1, F(n)=F(n - 1)+F(n - 2) (n \geq 3, n \in N \ast)$ 方程式的问题, 类似的问题比较多, 例如

- 共有 N 级台阶, 每次可以跨 2 个或者 1 个台阶, 总共有多少种走法, 比如有 3 级台阶, 可以三次都跨一个台阶, 或者先跨 2 个台阶, 再跨 1 个台阶, 也可以先跨一个台阶, 再跨 2 个台阶, 总共有三种方法
- 假设农场中成熟的母牛每年只会生一头小母牛, 并且永远不会死, 第一年农场有 1 头成熟的母牛, 从第二年开始, 母牛开始生小母牛, 每只小母牛 3 年后成熟了又可以生小母牛, 这样 N 年后农场的牛的数量多少, 比如第 1 年 1 头成熟母牛记为 a, 第 2 年 a 生了新的小母牛, 记为 b, 总牛数为 2, 第 3 年 a 生了新的小母牛, 记为 c, 总牛数为 3, 第 4 年 a 生了新的小母牛, 记为 d, 总牛数为 4。第 5 年 b 成熟了, a 和 b 分别生了新的小母牛, 总牛数为 6, 第 6 年 c 也成熟了, a、b 和 c 分别生了新的小母牛, 总牛数为 9

<!-- more -->

## 台阶问题

### 台阶问题解析

- 没有台阶, 那么 $F_0=0$
- 台阶只有 1 级, $F_1=1$
- 台阶有 2 级, $F_2=2$
- 有 N 级台阶, 需要考虑到最后上第 N 级台阶的情况
  - 可以是从 `N-2` 级台阶直接跨 2 级台阶
  - 可以是从 `N-1` 级台阶直接跨 1 级台阶

由上述的方法可以得到 $F(1)=1, F(2)=2, F(n)=F(n - 1)+F(n - 2) (n \geq 3, n \in N \ast)$

也就是

$$
\begin{cases}
  F_n = 0 & n=0 \\
  F_n = n & 1 \leq n \leq 2 \\
  F_n = F_{n-1} + F{n-2} & n \geq 3
\end{cases}
$$

这样递归方法和遍历方法均可以进行相应实现

将 将 $F_1=1, F_2=2, F_3=3, F_4=5$ 代入斐波那契数列矩阵计算出状态矩阵

$$
\left[
  \begin{array}{c}
    5 \\
    3
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
\right]^2
\times
\left[
  \begin{array}{c}
    2 \\
    1
  \end{array}
\right]
$$

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

### 台阶问题实现

这样只需要替换相应的变量即可实现

```java
public class Fibonacci {
    static int fibonacci(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 3) {
            return n;
        }
        // 递归
        return fibonacci(n - 1) + fibonacci(n - 2);
    }

    static int fibonacci2(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 3) {
            return n;
        }
        int tmp = 0;
        int pre = 1;
        int res = 2;
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
            return n;
        }
        int[][] base = {{1, 1}, {1, 0}};
        int[][] res = matrixPower(base, n - 2);
        // 根据分析得出第一项 F(2), F(1) 为 2, 1
        return 2 * res[0][0] + res[1][0];
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

### 台阶问题结果

```sh
0
1
2
3
5
8
13
21
34
55
---
0
1
1
3
5
8
13
21
34
55
---
0
1
1
3
5
8
13
21
34
55

Process finished with exit code 0
```

## 奶牛问题

### 奶牛问题解析

根据题目可以了解到这个逻辑比单纯的斐波那契数列更复杂, 由题目知道

- 第 1 年, 1 头牛
- 第 2 年, 2 头牛
- 第 3 年, 3 头牛
- 第 4 年, 4 头牛
- 第 5 年, 6 头牛
- 第 6 年, 9 头牛

复杂的题目从头开始往后缕思路都是个大坑

根据斐波那契数列的逻辑, 可以得到第 N 年的牛就是第 `N-1` 年的牛加上第 `N-3` 年所有成熟的牛生的小牛, 往前推类似

所以相应的模型为 $F(1)=1, F(2)=2, F(3)=3, F(n)=F(n - 1)+F(n - 3) (n \geq 4, n \in N \ast)$

也就是

$$
\begin{cases}
  F_n = 0 & n=0 \\
  F_n = n & 1 \leq n \leq 3 \\
  F_n = F_{n-1} + F{n-3} & n \geq 4
\end{cases}
$$

很明显与斐波那契数列方程式相差的是依赖的位置, 最后一位依赖倒数第二位和倒数第四位, 同理, 递归的方法与遍历的方法也仅需要根据方程式修改相应参数即可

由定义 $F_n=F_{n-1}+F_{n-3}$ 是一个三阶递推数列, 可以使用矩阵乘法的形式表示, 且状态矩阵为 $3 \times 3$ 的矩阵

当 $n \geq 4$ 时

$$
\left[
  \begin{array}{c}
    F_n \\
    F_{n-1} \\
    F_{n-2}
  \end{array}
\right]
=
\left[
  \begin{array}{ccc}
    a&b&c \\
    d&e&f \\
    g&h&j
  \end{array}
\right]
\times
\left[
  \begin{array}{c}
    F_{n-1} \\
    F_{n-2} \\
    F_{n-3}
  \end{array}
\right]
$$

代入 $F_1=1, F_2=2, F_3=3, F_4=4, F_5=6$ 计算出状态矩阵

$$
\left[
  \begin{array}{ccc}
    a&b&c \\
    d&e&f \\
    g&h&j
  \end{array}
\right]
=
\left[
  \begin{array}{ccc}
    1&1&0 \\
    0&0&1 \\
    1&0&0
  \end{array}
\right]
$$

所以

$$
\left[
  \begin{array}{c}
    F_n \\
    F_{n-1} \\
    F_{n-2}
  \end{array}
\right]
=
\left[
  \begin{array}{ccc}
    1&1&0 \\
    0&0&1 \\
    1&0&0
  \end{array}
\right]^{n-3}
\times
\left[
  \begin{array}{c}
    F_3 \\
    F_2 \\
    F_1
  \end{array}
\right]
\left[
  \begin{array}{ccc}
    1&1&0 \\
    0&0&1 \\
    1&0&0
  \end{array}
\right]^{n-3}
\times
\left[
  \begin{array}{c}
    3 \\
    2 \\
    1
  \end{array}
\right]
$$

### 奶牛问题实现

```java
public class Fibonacci {
    static int fibonacci(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 4) {
            return n;
        }
        // 递归
        return fibonacci(n - 1) + fibonacci(n - 3);
    }

    static int fibonacci2(int n) {
        if (n < 1) {
            return 0;
        }
        if (n < 4) {
            return n;
        }
        int tmp = 0;
        int tmp2 = 0;
        int pre = 2;
        int prepre = 1;
        // 第 3 项
        int res = 3;
        // 遍历
        for (int i = 4; i <= n; i ++) {
            tmp = res;
            tmp2 = pre;
            res = res + prepre;
            pre = tmp;
            prepre = tmp2;
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
        if (n < 4) {
            return n;
        }
        int[][] base = {{1, 1, 0}, {0, 0, 1}, {1, 0, 0}};
        int[][] res = matrixPower(base, n - 3);
        // 根据分析得出第一项为矩阵 3, 2, 1
        return 3 * res[0][0] + 2 * res[1][0] + res[2][0];
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

### 奶牛问题结果

```sh
0
1
2
3
4
6
9
13
19
28
---
0
1
2
3
4
6
9
13
19
28
---
0
1
2
3
4
6
9
13
19
28

Process finished with exit code 0
```

## 总结

个人感觉, 如果一个问题没有点明是类似斐波那契数列解法的话, 一开始入手还是比较困难的, 想法大概是, 如果顺着捋下去很难, 类似该题目, 每走一步都有很多结果, 那就直接从最后捋起来

这类的问题, 只要符合 $F_n=a \times F_{n-1} + b \times F_{n-2} + \cdots + k \times F_{n-i}$ 那么它就是一个 i 阶的递推式, 就可以使用上述的递归或者遍历的思路, 当然最快的方法是构建一个 $i \times i$ 的状态矩阵有关的矩阵乘法表达
