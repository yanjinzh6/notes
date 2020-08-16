---
title: 分治法
date: 2020-07-19 18:00:00
tags: '算法'
categories:
  - ['算法', '思想']
permalink: divide-and-conquer
photo:
mathjax: true
---

## 简介

分治法的设计思想是将无法着手解决的大问题分解成一系列规模较小的相同问题, 然后逐个解决小问题, 比如最大, 最小问题 (例如在一堆形状相同的物品中找出最重或最轻的那一个) , 矩阵乘法, 大整数乘法以及排序 (例如快速排序和归并排序)

许多高效算法都使用分治法的思想, 比如快速傅里叶变换算法和 `Karatsuba` 乘法算法

## 目的

- 通过分解问题, 使无法着手解决的大问题变成容易解决的小问题
- 通过减小问题的规模, 降低解决问题的复杂度 (或计算量)

<!-- more -->

## 基本思想

1. 分解: 将问题分解为若干个规模较小, 相互独立且与原问题形式相同的子问题, 确保各个子问题的解具有相同的子结构
2. 解决: 如果上一步分解得到的子问题可以解决, 则解决这些子问题, 否则, 对每个子问题使用和上一步相同的方法再次分解, 然后求解分解后的子问题, 这个过程可能是一个递归的过程
3. 合并: 将上一步解决的各个子问题的解通过某种规则合并起来, 得到原问题的解

采用递归方式的算法模式伪代码描述如下

```
T DivideAndConquer(P)
{
    if (P 可以直接解决)
    {
        T <- P 的结果;
        return T;
    }

    将 P 分解为子问题 [P1, P2,..., Pn];
    forEach (P Pi : [P1, P2,..., Pn])
    {
        // 递归解决子问题 Pi
        ti <- DivideAndConquer(Pi);
    }
    // 合并子问题的解
    T <- Merge(t1, t2,...,tn);
    return T;
}
```

难点: **如何将子问题分解, 并且将子问题的解合并出原始问题的解**

## 例子

分治法最难也是最灵活的部分就是对问题的分解和结果的合并, 对于一个未知的问题, 只要能找到对子问题的分解方式和结果的合并方式, 应用分治法就可以迎刃而解, 而在数学上, **只要是能用数学归纳法证明的问题, 一般都可以应用分治法解决**

### 快速傅里叶变换

- 分解思想: 计算 N 个采样点的离散傅里叶变换, 需要做 $N^2$ 次复数乘法, 按照奇偶关系分成两个 $\frac{N}{2}$ 个采样点的离散傅里叶变换, 只需要做 $(\frac{N}{2})^2 +(\frac{N}{2})^2 = \frac{N^2}{2}$ 次复数乘法, 做一次分解就使得计算量减少了一半
- 合并思想: 将两个 $\frac{N}{2}$ 点离散傅里叶变换的结果按照蝶形运算的位置关系重新排列成一个 N 点序列

### 快速排序算法

- 分解思想: 选择一个标兵数, 将待排序的序列分成两个子序列, 其中一个子序列中的数都小于标兵数, 另一个子序列中的数都大于标兵数, 然后分别对这两个子序列排序
- 合并思想: 将两个已经排序的子序列一前一后拼接在标兵数前后, 组成一个完整的有序序列

#### 普通快排

分解问题: 总序列 $[1, n]$ 的排序总是可以分解为子序列在区间 $[p, r] \subset [1, n]$ 的排序

```java
int partition(int[] arr, int left, int right) {
    int temp = arr[left];
    while (right > left) {
        // 先判断基准数和后面的数依次比较
        while (temp <= arr[right] && left < right) {
            --right;
        }
        // 当基准数大于了 arr[right], 则填坑
        if (left < right) {
            arr[left] = arr[right];
            ++left;
        }
        // 现在是 arr[right] 需要填坑了
        while (temp >= arr[left] && left < right) {
            ++left;
        }
        if (left < right) {
            arr[right] = arr[left];
            --right;
        }
    }
    arr[left] = temp;
    return left;
}
void quickSort(int[] arr, int left, int right) {
    if (arr == null || left >= right || arr.length <= 1)
        return;
    int mid = partition(arr, left, right);
    quickSort(arr, left, mid);
    quickSort(arr, mid + 1, right);
}
```

最好情况下时间复杂度为 $O(nlogn)$, 如果每次分区后都出现子序列的长度一个为 1 一个为 $n-1$, 那会导致 $T(n) = O(n) + T(1) + T(n-1) = O(n) + T(n-1)$, 最坏的情况下时间复杂度为 $O(n^2)$, 可以采用三数取中法优化避免最坏的情况, 通过使用左端, 右端和中心位置上的三个元素的中值作为基准元消除预排序输入的不好情形, 并且减少快排大约 `5%` 的比较次数

#### 三数取中法快排

```java
void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

int partition(int[] arr, int left, int right) {
    // 采用三数中值分割法
    int mid = left + (right - left) / 2;
    // 保证左端较小
    if (arr[left] > arr[right])
        swap(arr, left, right);
    // 保证中间较小
    if (arr[mid] > arr[right])
        swap(arr, mid, right);
    // 保证中间最小, 左右最大
    if (arr[mid] > arr[left])
        swap(arr, left, mid);
    int pivot = arr[left];
    while (right > left) {
        // 先判断基准数和后面的数依次比较
        while (pivot <= arr[right] && left < right) {
            --right;
        }
        // 当基准数大于了 arr[right], 则填坑
        if (left < right) {
            arr[left] = arr[right];
            ++left;
        }
        // 现在是 arr[right] 需要填坑了
        while (pivot >= arr[left] && left < right) {
            ++left;
        }
        if (left < right) {
            arr[right] = arr[left];
            --right;
        }
    }
    arr[left] = pivot;
    return left;
}

void quickSort(int[] arr, int left, int right) {
    if (arr == null || left >= right || arr.length <= 1)
        return;
    int mid = partition(arr, left, right);
    quickSort(arr, left, mid);
    quickSort(arr, mid + 1, right);
}
```

### Karatsuba 乘法算法

两个 n 位大整数相乘, 普通乘法算法的时间复杂度一般是 $O ( n^2 )$, 但是 `Anatolii Alexeevitch Karatsuba` 博士在 1960 年提出了一种时间复杂度是 $O(n^{\log_{2}3})$ 的快速乘法算法, 这就是 Karatsuba 乘法算法

- 分解思想: 将 n 位大数分成两部分：$a + b$ , 其中 a 是整数幂, 然后利用乘法的分解公式：$( a + b )( c + d )= ac + ad + bc + bd$ , 将其分解为四次小规模大数的乘法计算, 然后利用一个小技巧将其化解成三次乘法和少量移位操作
- 合并思想: 用几次加法对小规模乘法的结果进行求和, 得到原始问题的解

#### 推导

假如有两个 n 位的 M 进制大整数 x , y , 利用一个小于 n 的正数 k  (通常 k 的取值为 n /2 左右) , 将 x 和 y 分解为两个部分

$$
x = x_1 M^k + x_0 \\\\
y = y_1 M^k + y_0
$$

则 x 和 y 的乘积可计算为

$$
xy = (x_1 M^k + x_0)(y_1 M^k + y_0) = x_1 y_1 M^2k + (x_1 y_0 + x_0 y_1) M^k + x_1 y_0
$$

这样就将 x 和 y 的乘法计算转化成四次较小规模的乘法计算和少量的加法计算, 其中 $M^2k$ 和 $M^k$ 的计算都可以通过移位高效地处理, 不过上述操作还可以继续优化, 我们令 $z_0 = x_0 y_0 , z_1 = x_1 y_0 + x_0 y_1 , z_2 = x_1 y_1$, 则 $xy$ 的乘积可表示为

$$
xy = z_2 M^2k + z_1 M^k + z_0
$$

计算 $z_1$ 需要两次乘法, 对 $z_1$ 的计算可以优化为

$$
z_1 = (x_1 + x_0)(y_1 + y_0) - x_1 y_1 - x_0 y_0 = (x_1 + x_0)(y_1 + y_0) - z_2 - z_0
$$

由于 $z_0$ 和 $z_2$ 都已经计算过了, 因此就只需一次乘法, 辅助两次加法和两次减法即可计算出 $z_1$

#### 根据推导计算

通过简单实例计算 $12345 * 6789$, 我们让 $a = 12, b = 345$, 同时 $c = 6, d = 789$

即

$$
12345=12·1000+345 \\\\
6789=6·1000+789
$$

计算得出

$$
z_2 = a*c = 12 × 6 = 72 \\\\
z_0 = b*d = 345 × 789 = 272205 \\\\
z_1 = ((a+b)*(c+d) - a*c - b*d) \\\\
    = (12 + 345) × (6 + 789) − z_2 − z_0 = 283815 − 72 − 272205 = 11538
$$

最终结果

$$
result = z_2 · 10^{2*3} + z_1 · 10^3 + z_0 \\\\
result = 72 · 10^6 + 11538 · 10^3 + 272205 = 83810205
$$

#### 简单代码

这里使用 `long` 类型参数, 当数字足够大的情况下需要使用到 `BigInteger` 类型

```java
/**
 * Karatsuba乘法
 */
long karatsuba(long num1, long num2){
    //递归终止条件
    if(num1 < 10 || num2 < 10) return num1 * num2;

    // 计算拆分长度
    int size1 = String.valueOf(num1).length();
    int size2 = String.valueOf(num2).length();
    int halfN = Math.max(size1, size2) / 2;

    /* 拆分为a, b, c, d */
    long a = Long.valueOf(String.valueOf(num1).substring(0, size1 - halfN));
    long b = Long.valueOf(String.valueOf(num1).substring(size1 - halfN));
    long c = Long.valueOf(String.valueOf(num2).substring(0, size2 - halfN));
    long d = Long.valueOf(String.valueOf(num2).substring(size2 - halfN));

    // 计算z2, z0, z1, 此处的乘法使用递归
    long z2 = karatsuba(a, c);
    long z0 = karatsuba(b, d);
    long z1 = karatsuba((a + b), (c + d)) - z0 - z2;

    return (long)(z2 * Math.pow(10, (2*halfN)) + z1 * Math.pow(10, halfN) + z0);
}
```

`Karatsuba` 算法是比较简单的递归乘法, 把输入拆分成 `2` 部分, 不过对于更大的数, 可以把输入拆分成 `3` 部分甚至 `4` 部分, 拆分为 `3` 部分时, 可以使用 `Toom-Cook 3-way` 乘法, 复杂度降低到 $O(n^1.465)$, 拆分为 `4` 部分时, 使用 `Toom-Cook 4-way` 乘法, 复杂度进一步下降到 $O(n^1.404)$, 对于更大的数字, 可以拆成 `100` 段, 使用`快速傅里叶变换 FFT`, 复杂度接近线性, 大约是 $O(n^1.149)$, 可以看出, 分割越大, 时间复杂度就越低, 但是所要计算的中间项以及合并最终结果的过程就会越复杂, 开销会增加, 因此分割点上升, 对于公钥加密, 暂时用不到太大的整数, 所以使用 `Karatsuba` 就合适

#### BigInteger 的乘法实现

目前最著名的高精度整数运算库是 GNU 的 [GMP (The GNU MP Bignum Library)](https://gmplib.org/), 可以用于任意精度的数学运算, 包括有符号整数, 有理数和浮点数, 它本身并没有精度限制, 只取决于机器的硬件情况, 许多著名的计算机代数系统如 Axiom, Maple, Mathematica, Maxima 等的底层高精度整数运算都是基于 GMP 实现的

设 n 为乘数的位数, 时间复杂度如下

- 普通算法: $O(n^2)$
- Karatsuba 乘法: $O(n^{log_2 3})$
- Toom-3 乘法: $O(n^{log_3 5})$
- 复数域上的 FFT:
  - $O(n {\log^* n})$
  - $log^* n = log n(log log n)(log log log n) ···$
- 有限域上的 FFT: $O(n(log n)(log log n))$, 时间复杂度已经相当接近线性

但是这些乘法算法中复杂度较低的算法往往有较大的常数因子, 因此**如果乘数的位数较少, 普通乘法反而是最快的**, 所以实用中常常将这些不同的乘法算法结合起来使用, 每次做乘法时都根据相乘两数的大小动态地选择具体采用哪一种算法, 而每种算法的最佳适用范围往往依赖于具体实现和硬件环境, 因此一般直接通过实验来确定

- Java 7 中直接用二重循环直接相乘, 见[源码第 1165 行](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/00cd9dc3c2b5/src/share/classes/java/math/BigInteger.java#l1165)
- Java 8 中, 根据两个因数的大小区分, 见[源码第 1464 行](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/tip/src/share/classes/java/math/BigInteger.java#l1464)
  - 当两个因数均小于 $2^{32\times 80}$ 时, 用二重循环直接相乘, 复杂度为 $O(n^2)$, n 为因数位数
  - 当两个因数均小于 $2^{32\times 240}$ 时, 采用 `Karatsuba algorithm`, 其复杂度为 $O(n^{\log_2 3}) \approx O(n^{1.585})$, n 为因数位数
  - 否则, 采用 `Toom-Cook multiplication`, 其复杂度为 $O(n^{\log_3 5}) \approx O(n^{1.465})$, n 为因数位数

#### Java 8 源码

```java
/**
 * The threshold value for using squaring code to perform multiplication
 * of a {@code BigInteger} instance by itself.  If the number of ints in
 * the number are larger than this value, {@code multiply(this)} will
 * return {@code square()}.
 */
private static final int MULTIPLY_SQUARE_THRESHOLD = 20;
/**
 * The threshold value for using Karatsuba multiplication.  If the number
 * of ints in both mag arrays are greater than this number, then
 * Karatsuba multiplication will be used.   This value is found
 * experimentally to work well.
 */
private static final int KARATSUBA_THRESHOLD = 80;

/**
 * The threshold value for using 3-way Toom-Cook multiplication.
 * If the number of ints in each mag array is greater than the
 * Karatsuba threshold, and the number of ints in at least one of
 * the mag arrays is greater than this threshold, then Toom-Cook
 * multiplication will be used.
 */
private static final int TOOM_COOK_THRESHOLD = 240;

// java.math.BigInteger#multiply(java.math.BigInteger)
/**
 * Returns a BigInteger whose value is {@code (this * val)}.
 *
 * @implNote An implementation may offer better algorithmic
 * performance when {@code val == this}.
 *
 * @param  val value to be multiplied by this BigInteger.
 * @return {@code this * val}
 */
public BigInteger multiply(BigInteger val) {
    if (val.signum == 0 || signum == 0)
        return ZERO;

    int xlen = mag.length;

    if (val == this && xlen > MULTIPLY_SQUARE_THRESHOLD) {
        return square();
    }

    int ylen = val.mag.length;

    if ((xlen < KARATSUBA_THRESHOLD) || (ylen < KARATSUBA_THRESHOLD)) {
        int resultSign = signum == val.signum ? 1 : -1;
        if (val.mag.length == 1) {
            return multiplyByInt(mag,val.mag[0], resultSign);
        }
        if (mag.length == 1) {
            return multiplyByInt(val.mag,mag[0], resultSign);
        }
        int[] result = multiplyToLen(mag, xlen,
                                        val.mag, ylen, null);
        result = trustedStripLeadingZeroInts(result);
        return new BigInteger(result, resultSign);
    } else {
        if ((xlen < TOOM_COOK_THRESHOLD) && (ylen < TOOM_COOK_THRESHOLD)) {
            return multiplyKaratsuba(this, val);
        } else {
            return multiplyToomCook3(this, val);
        }
    }
}
```

```java
// java.math.BigInteger#multiplyKaratsuba
/**
 * Multiplies two BigIntegers using the Karatsuba multiplication
 * algorithm.  This is a recursive divide-and-conquer algorithm which is
 * more efficient for large numbers than what is commonly called the
 * "grade-school" algorithm used in multiplyToLen.  If the numbers to be
 * multiplied have length n, the "grade-school" algorithm has an
 * asymptotic complexity of O(n^2).  In contrast, the Karatsuba algorithm
 * has complexity of O(n^(log2(3))), or O(n^1.585).  It achieves this
 * increased performance by doing 3 multiplies instead of 4 when
 * evaluating the product.  As it has some overhead, should be used when
 * both numbers are larger than a certain threshold (found
 * experimentally).
 *
 * See:  http://en.wikipedia.org/wiki/Karatsuba_algorithm
 */
private static BigInteger multiplyKaratsuba(BigInteger x, BigInteger y) {
    int xlen = x.mag.length;
    int ylen = y.mag.length;

    // The number of ints in each half of the number.
    int half = (Math.max(xlen, ylen)+1) / 2;

    // xl and yl are the lower halves of x and y respectively,
    // xh and yh are the upper halves.
    BigInteger xl = x.getLower(half);
    BigInteger xh = x.getUpper(half);
    BigInteger yl = y.getLower(half);
    BigInteger yh = y.getUpper(half);

    BigInteger p1 = xh.multiply(yh);  // p1 = xh*yh
    BigInteger p2 = xl.multiply(yl);  // p2 = xl*yl

    // p3=(xh+xl)*(yh+yl)
    BigInteger p3 = xh.add(xl).multiply(yh.add(yl));

    // result = p1 * 2^(32*2*half) + (p3 - p1 - p2) * 2^(32*half) + p2
    BigInteger result = p1.shiftLeft(32*half).add(p3.subtract(p1).subtract(p2)).shiftLeft(32*half).add(p2);

    if (x.signum != y.signum) {
        return result.negate();
    } else {
        return result;
    }
}
```

```java
// java.math.BigInteger#multiplyToomCook3
/**
 * Multiplies two BigIntegers using a 3-way Toom-Cook multiplication
 * algorithm.  This is a recursive divide-and-conquer algorithm which is
 * more efficient for large numbers than what is commonly called the
 * "grade-school" algorithm used in multiplyToLen.  If the numbers to be
 * multiplied have length n, the "grade-school" algorithm has an
 * asymptotic complexity of O(n^2).  In contrast, 3-way Toom-Cook has a
 * complexity of about O(n^1.465).  It achieves this increased asymptotic
 * performance by breaking each number into three parts and by doing 5
 * multiplies instead of 9 when evaluating the product.  Due to overhead
 * (additions, shifts, and one division) in the Toom-Cook algorithm, it
 * should only be used when both numbers are larger than a certain
 * threshold (found experimentally).  This threshold is generally larger
 * than that for Karatsuba multiplication, so this algorithm is generally
 * only used when numbers become significantly larger.
 *
 * The algorithm used is the "optimal" 3-way Toom-Cook algorithm outlined
 * by Marco Bodrato.
 *
 *  See: http://bodrato.it/toom-cook/
 *       http://bodrato.it/papers/#WAIFI2007
 *
 * "Towards Optimal Toom-Cook Multiplication for Univariate and
 * Multivariate Polynomials in Characteristic 2 and 0." by Marco BODRATO;
 * In C.Carlet and B.Sunar, Eds., "WAIFI'07 proceedings", p. 116-133,
 * LNCS #4547. Springer, Madrid, Spain, June 21-22, 2007.
 *
 */
private static BigInteger multiplyToomCook3(BigInteger a, BigInteger b) {
    int alen = a.mag.length;
    int blen = b.mag.length;

    int largest = Math.max(alen, blen);

    // k is the size (in ints) of the lower-order slices.
    int k = (largest+2)/3;   // Equal to ceil(largest/3)

    // r is the size (in ints) of the highest-order slice.
    int r = largest - 2*k;

    // Obtain slices of the numbers. a2 and b2 are the most significant
    // bits of the numbers a and b, and a0 and b0 the least significant.
    BigInteger a0, a1, a2, b0, b1, b2;
    a2 = a.getToomSlice(k, r, 0, largest);
    a1 = a.getToomSlice(k, r, 1, largest);
    a0 = a.getToomSlice(k, r, 2, largest);
    b2 = b.getToomSlice(k, r, 0, largest);
    b1 = b.getToomSlice(k, r, 1, largest);
    b0 = b.getToomSlice(k, r, 2, largest);

    BigInteger v0, v1, v2, vm1, vinf, t1, t2, tm1, da1, db1;

    v0 = a0.multiply(b0);
    da1 = a2.add(a0);
    db1 = b2.add(b0);
    vm1 = da1.subtract(a1).multiply(db1.subtract(b1));
    da1 = da1.add(a1);
    db1 = db1.add(b1);
    v1 = da1.multiply(db1);
    v2 = da1.add(a2).shiftLeft(1).subtract(a0).multiply(
            db1.add(b2).shiftLeft(1).subtract(b0));
    vinf = a2.multiply(b2);

    // The algorithm requires two divisions by 2 and one by 3.
    // All divisions are known to be exact, that is, they do not produce
    // remainders, and all results are positive.  The divisions by 2 are
    // implemented as right shifts which are relatively efficient, leaving
    // only an exact division by 3, which is done by a specialized
    // linear-time algorithm.
    t2 = v2.subtract(vm1).exactDivideBy3();
    tm1 = v1.subtract(vm1).shiftRight(1);
    t1 = v1.subtract(v0);
    t2 = t2.subtract(t1).shiftRight(1);
    t1 = t1.subtract(tm1).subtract(vinf);
    t2 = t2.subtract(vinf.shiftLeft(1));
    tm1 = tm1.subtract(t2);

    // Number of bits to shift left.
    int ss = k*32;

    BigInteger result = vinf.shiftLeft(ss).add(t2).shiftLeft(ss).add(t1).shiftLeft(ss).add(tm1).shiftLeft(ss).add(v0);

    if (a.signum != b.signum) {
        return result.negate();
    } else {
        return result;
    }
}
```
