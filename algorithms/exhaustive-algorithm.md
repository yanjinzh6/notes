---
title: 穷举法
date: 2020-07-19 22:00:00
tags: '算法'
categories:
  - ['算法', '思想']
permalink: exhaustive-algorithm
photo:
mathjax: true
---

## 简介

数学上也把穷举法称为枚举法, 就是在一个由有限个元素构成的集合中, 将所有元素一一枚举研究的方法

## 解空间

全部可能的候选解的一个约束范围, 确定问题的解就在这个范围内, 将搜索策略应用到这个约束范围就可以找到问题的解, 解空间的结构可能是线性表、集合、树或者图, 有时候, 这个空间内的对象不是问题的解, 而是一些被称为状态的对象, 通过对状态的计算和处理, 可以间接地得到问题的解, 这样的搜索空间也常被理解为状态空间

穷举并不是漫无目的地乱找, 它是一种在有限的解空间 (解空间至少在理论上是有限的) 内按照一定的策略进行查找的思想

<!-- more -->

## 思想

对解空间内的候选解按某种顺序进行逐一枚举和检验, 并根据问题给定的条件从中找出那些符合要求的候选解作为问题的解, 穷举法一般可以找出解空间中所有正确的解, 如果给定最优解的判断条件, 穷举法也可以用于求解最优解问题

一般来说, 只要一个问题有其他更好的解决方法, 通常不会选择穷举法

但是穷举法在算法设计模式中占有非常重要的地位, 它还是很多问题的唯一解决方法

穷举法虽然思想简单, 但是设计一个解决特定问题的穷举法实现却并不简单, 难点如下

- 解空间或状态空间的定义没有具体的模式, 不同问题的解空间形式上差异巨大
- 针对不同问题都要选择不同的搜索算法, 有很多问题的搜索算法并不直观, 需要对问题做细致的分析才能得到

## 步骤

1. 确定问题的解 (或状态) 的定义, 解空间的范围以及正确解的判定条件
2. 根据解空间的特点选择搜索策略, 一一检验解空间中的候选解是否正确, 必要时可辅助一些剪枝算法, 排除一些明显不可能是正确解的检验过程, 提高穷举的效率

## 穷举解空间的策略

搜索算法的设计策略, 简单的问题可以用通用的搜索算法, 比如 0-1 背包问题的解空间可以用排列组合算法得到, 复杂的问题需要根据实际情况设计搜索算法

### 盲目搜索算法

广度优先搜索和深度优先搜索是两种常用的盲目搜索算法, 这种搜索算法只根据问题的规模, 按照广度优先和深度优先的原则搜索解空间内的每一个状态

- 广度优先算法因为需要额外的存储空间, 因此在设计算法时要考虑此额外空间的规模
- 深度优先算法在搜索过程中容易陷入状态循环, 导致在一个没有解的子树上 "死循环" , 一般需要做状态循环的判断和避免

### 启发式搜索算法

当问题的规模达到一定的程度, 盲目搜索算法就会因为低效而被排斥, 启发性搜索可以利用搜索过程中出现的额外信息直接跳过一些状态, 避免盲目的、机械式的搜索, 就可以加快搜索算法的收敛

- 知道解空间的状态分布呈现正态分布的特征, 可以从分布中间值开始向两边搜索, 因为在中间值附近出现最优解的概率更高
- 存在一个状态评估函数, 可以对每个状态节点能演化出解的可能性进行评估, 搜索过程中根据这种可能性对待搜索的状态节点排序
- 在某一个层面的搜索能应用贪婪策略, 优先选择与贪婪策略符合的状态节点进行搜索

### 剪枝策略

穷举搜索解空间最优解时, 有一些状态节点根据问题提供的信息明确地被判定为不可能演化出最优解, 就可以跳过此状态节点的遍历, 这将极大地提高算法的执行效率

为了避免某些循环的状态链, 也需要使用剪枝策略跳出循环遍历的状态, 需要找到一种算法对状态计算校验值, 通过比较校验值判断是否是已经处理过的状态节点, 例如对状态 A 搜索得到子状态 B, 对状态 B 搜索得到子状态 C, 对状态 C 搜索又可得到子状态 A 的情况, 就会使得搜索算法陷入 "死循环" , 这时候应该排除掉状态 A

### 搜索算法的评估和收敛

收敛原则就是只要能找到一个比较好的解就返回 (不求最好) , 根据解的评估判断是否需要继续下一次搜索

很多问题, 当规模大到一定程度时, 使用穷举法就只具有理论上的可行性, 对于那些只能使用穷举法, 但是问题规模又大到无法对解空间进行完整的搜索, 这时候就需要对搜索算法进行评估, 并确定一些收敛原则

比如国际象棋和围棋的求解算法, 不可能在可等待的时间内搜索整个解空间的最优解, 所以此类搜索算法通常都通过一个搜索深度参数来控制搜索算法的收敛, 当搜索到指定的深度时 (相当于走了若干步棋) 就返回当前已经找到的最好的结果

## 例子

有一个由字符组成的等式：WWWDOT - GOOGLE = DOTCOM , 每个字符代表一个 0～9 之间的数字, WWWDOT、GOOGLE 和 DOTCOM 都是合法的数字, 不能以 0 开头, 请找出一组字符和数字的对应关系, 使它们互相替换, 并且替换后的数字能够满足等式

从穷举法的角度看, 这是个组合问题, 如果不考虑 0 开头数字的情况, 这样的组合应该有 10×9×8×7×6×5×4×3×2=3628800 种组合, 在这样的数量级上使用穷举法, 计算机处理起来应该没有压力

定义一个可变化的字符元素列表, 每个字符元素包含 3 个属性, 分别是字母本身、字母代表的数字以及是否是数字的最高位 (根据题意, 最高位不能是 0, 所以要特别对待)

```java
@AllArgsConstructor
public class TagCharItem {
    public char c;
    public int value;
    public boolean leading;
}
```

初始化列表

```java
TagCharItem[] tagCharItem = {
    new TagCharItem('W', -1, true),
    new TagCharItem('D', -1, true),
    new TagCharItem('O', -1, false),
    new TagCharItem('T', -1, false),
    new TagCharItem('G', -1, true),
    new TagCharItem('L', -1, false),
    new TagCharItem('E', -1, false),
    new TagCharItem('C', -1, false),
    new TagCharItem('M', -1, false)
};
```

因为这是一个组合问题, 两个字母不能被指定为相同的数字, 这就需要对每个数字做一个标识, 当这个数字已经被某个字符 "占用" 时, 其他字符不能再使用这个数字

```java
public class TagCharValue {
  public bool used;
  public int value;
}
```

初始化列表

```java
TagCharValue[] tagCharValues = {
    new TagCharValue(false, 0),
    new TagCharValue(false, 1),
    new TagCharValue(false, 2),
    new TagCharValue(false, 3),
    new TagCharValue(false, 4),
    new TagCharValue(false, 5),
    new TagCharValue(false, 6),
    new TagCharValue(false, 7),
    new TagCharValue(false, 8),
    new TagCharValue(false, 9)
};
```

采用递归的方式进行组合枚举, 按照 charItem 列表中的顺序, 逐个对每个字符进行数字遍历

根据题目要求, W、G 和 D 这 3 个字符不能是 0, 因此枚举过程中对这 3 个字符是 0 的情况进行剪枝, `isValueValid()` 函数就是评估函数, 通过剪枝操作, 被调用的次数由理论上的 3628800 次减少为 2540160, 减少了约 30% 的计算判断, index 参数标识字符索引, 每次递归调用对索引为 index 的字符进行数字遍历, 当 index 等于字符个数时, 表示所有的字符都已经指定了对应的数字, 可以调用 callback 进行结果判断, SearchingResult() 函数可以作为此类问题的通用搜索框架, 只需指定不同的 CHAR_ITEM 和 CHAR_VALUE 参数, 以及结果处理回调 callback 即可

```java
/**
 * 穷举法
 */
public class Exhaustive {
    /**
      * 使用递归方式进行组合枚举
      *
      * @param tagCharItems  字符串对应数字列表
      * @param tagCharValues 数字占用列表
      * @param index         索引位置
      */
    public void searchingResult(TagCharItem[] tagCharItems, TagCharValue[] tagCharValues, int index) {
        if (index == tagCharItems.length) {
            isOK(tagCharItems);
            return;
        }

        for (int i = 0; i < tagCharValues.length; i++) {
            if (isValueValid(tagCharItems[index], tagCharValues[i])) {
                tagCharValues[i].used = true;
                tagCharItems[index].value = tagCharValues[i].value;

                searchingResult(tagCharItems, tagCharValues, index + 1);

                tagCharValues[i].used = false;
            }
        }
    }

    /**
      * 判定数据是否有效, 是否可以被赋值
      *
      * @param tagCharItem  字符串对应数字
      * @param tagCharValue 数字占用
      * @return 是否可以被赋值
      */
    private boolean isValueValid(TagCharItem tagCharItem, TagCharValue tagCharValue) {
        if (tagCharItem.leading) {
            if (!tagCharValue.used) {
                if (tagCharValue.value == 0) {
                    return false;
                } else {
                    return true;
                }
            } else {
                return false;
            }
        } else {
            if (!tagCharValue.used) {
                return true;
            } else {
                return false;
            }
        }
    }


    /**
      * 验证数据
      *
      * @param tagCharItems 字符串对应数字列表
      */
    private void isOK(TagCharItem[] tagCharItems) {
        try {
            String WWWDOT = "WWWDOT";
            WWWDOT = strToNum(WWWDOT, tagCharItems);
            int _WWWDOT = Integer.parseInt(WWWDOT);

            String GOOGLE = "GOOGLE";
            GOOGLE = strToNum(GOOGLE, tagCharItems);
            int _GOOGLE = Integer.parseInt(GOOGLE);

            String DOTCOM = "DOTCOM";
            DOTCOM = strToNum(DOTCOM, tagCharItems);
            int _DOTCOM = Integer.parseInt(DOTCOM);

            if (_WWWDOT - _GOOGLE == _DOTCOM) {
                System.out.println("WWWDOT = " + _WWWDOT + ",GOOGLE = " + _GOOGLE + ",DOTCOM = " + _DOTCOM);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
      * 将字符串转换为数字
      *
      * @param str          字符串
      * @param tagCharItems 字符串对应数字列表
      * @return
      */
    private String strToNum(String str, TagCharItem[] tagCharItems) {
        for (int i = 0; i < tagCharItems.length; i++) {
            str = str.replace(String.valueOf(tagCharItems[i].c), String.valueOf(tagCharItems[i].value));
        }
        return str;
    }
}
```

对整个解空间搜索得到答案

```sh
WWWDOT = 777589,GOOGLE = 188103,DOTCOM = 589486
WWWDOT = 777589,GOOGLE = 188106,DOTCOM = 589483

Process finished with exit code 0
```
