---
title: Java 集合列表
date: 2020-05-01 8:30:00
tags: 'Java'
categories:
  - ['开发', 'Java', '基础']
permalink: java-collection-describetion
---

## 简介

Java 的集合类被定义在 Java.util 包中, 主要有 4 种集合, 分别为 List, Queue, Set 和 Map

## List

List 是有序的 Collection

### ArrayList: 基于数组实现, 增删慢, 查询快, 线程不安全

ArrayList 内部数据结构基于数组实现, 提供了对 List 的增加 (add) , 删除 (remove) 和访问 (get) 功能, 对元素必须连续存储

- 优点: 适合随机查找
- 缺点: 当需要在 ArrayList 的中间位置插入或者删除元素时, 需要将待插入或者删除的节点后的所有元素进行移动, 其修改代价较高

ArrayList 不需要在定义时指定数组的长度, 在数组长度不能满足存储要求时, 会创建一个新的更大的数组并将数组中已有的数据复制到新的数组中

### Vector: 基于数组实现, 增删慢, 查询快, 线程安全

Vector 的数据结构和 ArrayList 一样, 都是基于数组实现的, 不同的是 Vector 支持线程同步, 即同一时刻只允许一个线程对 Vector 进行写操作 (新增, 删除, 修改) , 以保证多线程环境下数据的一致性, 但需要频繁地对 Vector 实例进行加锁和释放锁操作, 因此, Vector 的读写效率在整体上比 ArrayList 低

### LinkedList: 基于双向链表实现, 增删快, 查询慢, 线程不安全

LinkedList 采用双向链表结构存储元素

- 优点: 进行插入和删除操作时, 只需在对应的节点上插入或删除元素, 并将上一个节点元素的下一个节点的指针指向该节点即可, 数据改动较小, 因此随机插入和删除效率很高
- 缺点: 进行随机访问时, 需要从链表头部一直遍历到该节点为止, 因此随机访问速度很慢

LinkedList 还提供了在 List 接口中未定义的方法, 用于操作链表头部和尾部的元素, 因此有时可以被当作堆栈, 队列或双向队列使用

<!-- more -->

## Queue

Queue 是队列结构, 常用队列如下

- ArrayBlockingQueue: 基于数组数据结构实现的有界阻塞队列
- LinkedBlockingQueue: 基于链表数据结构实现的有界阻塞队列
- PriorityBlockingQueue: 支持优先级排序的无界阻塞队列
- DelayQueue: 支持延迟操作的无界阻塞队列
- SynchronousQueue: 用于线程同步的阻塞队列
- LinkedTransferQueue: 基于链表数据结构实现的无界阻塞队列
- LinkedBlockingDeque: 基于链表数据结构实现的双向阻塞队列

## Set

Set 适用于存储无序且值不相等的元素, 对象的相等性在本质上是对象的 HashCode 值相同, Java 依据对象的内存地址计算出对象的 HashCode 值, 想要比较两个对象是否相等, 必须同时覆盖对象的 hashCode 方法和 equals 方法, 并且 hashCode 方法和 equals 方法的返回值必须相同

### HashSet: HashTable 实现, 无序

HashSet 存放的是散列值, 它是按照元素的散列值来存取元素的, 元素的散列值是通过元素的 hashCode 方法计算得到的, HashSet 首先判断两个元素的散列值是否相等, 如果散列值相等, 则接着通过 equals 方法比较, 如果 equls 方法返回的结果也为 true, HashSet 就将其视为同一个元素；如果 equals 方法返回的结果为 false, HashSet 就不将其视为同一个元素

### TreeSet: 二叉树实现

TreeSet 基于二叉树的原理对新添加的对象按照指定的顺序排序 (升序, 降序) , 每添加一个对象都会进行排序, 并将对象插入二叉树指定的位置

`Integer` 和 `String` 等基础对象类型可以直接根据 TreeSet 的默认排序进行存储, 而自定义的数据类型必须实现 `Comparable` 接口, 并且覆写其中的 `compareTo` 函数才可以按照预定义的顺序存储, 若覆写 `compare` 函数, 则在升序时在 `this.对象` 小于指定对象的条件下返回 `-1`, 在降序时在 `this.对象` 大于指定对象的条件下返回 `1`

### LinkHashSet: HashTable 实现数据存储, 双向链表记录顺序

LinkedHashSet 在底层使用 LinkedHashMap 存储元素, 它继承了 HashSet, 所有的方法和操作都与 HashSet 相同, 因此 LinkedHashSet 的实现比较简单, 只提供了 4 个构造方法, 并通过传递一个标识参数调用父类的构造器, 在底层构造一个 LinkedHashMap 来记录数据访问, 其他相关操作与父类 HashSet 相同, 直接调用父类 HashSet 的方法即可

## Map

### HashMap: 数组 + 链表存储数据, 线程不安全

HashMap 基于键的 HashCode 值唯一标识一条数据, 同时基于键的 HashCode 值进行数据的存取, 因此可以快速地更新和查询数据, 但其每次遍历的顺序无法保证相同, HashMap 的 key 和 value 允许为 null

HashMap 是非线程安全的, 即在同一时刻有多个线程同时写 HashMap 时将可能导致数据的不一致, 如果需要满足线程安全的条件, 则可以用 Collections 的 synchronizedMap 方法使 HashMap 具有线程安全的能力, 或者使用 ConcurrentHashMap

HashMap 内部是一个数组, 数组中的每个元素都是一个单向链表, 链表中的每个元素都是嵌套类 Entry 的实例, Entry 实例包含 4 个属性: key, value, hash 值和用于指向单向链表下一个元素的 next

常用参数

- `capacity: 当前数组的容量, 默认为` `16`, 可以扩容, 扩容后数组的大小为当前的两倍, 因此该值始终为 `2n`
- loadFactor: 负载因子, 默认为 `0.75`
- threshold: 扩容的阈值, 其值等于 `capacity×loadFactor`

HashMap 在查找数据时, 根据 HashMap 的 Hash 值可以快速定位到数组的具体下标, 但是在找到数组下标后需要对链表进行顺序遍历直到找到需要的数据, 时间复杂度为 `O(n)`

为了减少链表遍历的开销, Java 8 对 HashMap 进行了优化, 将数据结构修改为数组 + 链表或红黑树, 在链表中的元素超过 8 个以后, HashMap 会将链表结构转换为红黑树结构以提高查询效率, 因此其时间复杂度为 `O(log N)`

### ConcurrentHashMap: 分段锁实现, 线程安全

与 HashMap 不同, ConcurrentHashMap 采用分段锁的思想实现并发操作, 因此是线程安全的, ConcurrentHashMap 由多个 Segment 组成 (Segment 的数量也是锁的并发度) , 每个 Segment 均继承自 ReentrantLock 并单独加锁, 所以每次进行加锁操作时锁住的都是一个 Segment, 这样只要保证每个 Segment 都是线程安全的, 也就实现了全局的线程安全

在 ConcurrentHashMap 中有个 concurrencyLevel 参数表示并行级别, 默认是 `16`, 也就是说 ConcurrentHashMap 默认由 `16` 个 Segments 组成, 在这种情况下最多同时支持 `16` 个线程并发执行写操作, 只要它们的操作分布在不同的 Segment 上即可, 并行级别 concurrencyLevel 可以在初始化时设置, 一旦初始化就不可更改, ConcurrentHashMap 的每个 Segment 内部的数据结构都和 HashMap 相同

Java 8 在 ConcurrentHashMap 中引入了红黑树

### HashTable: 线程安全

HashTable 是遗留类, 很多映射的常用功能都与 HashMap 类似, 不同的是它继承自 Dictionary 类, 并且是线程安全的, 同一时刻只有一个线程能写 HashTable, 并发性不如 ConcurrentHashMap

### TreeMap: 基于二叉树数据结构

TreeMap 基于二叉树数据结构存储数据, 同时实现了 SortedMap 接口以保障元素的顺序存取, 默认按键值的升序排序, 也可以自定义排序比较器

TreeMap 常用于实现排序的映射列表, 在使用 TreeMap 时其 key 必须实现 Comparable 接口或采用自定义的比较器, 否则会抛出 java.lang.ClassCastException 异常

### LinkedHashMap: 基于 HashTable 数据结构, 使用链表保存插入顺序

LinkedHashMap 为 HashMap 的子类, 其内部使用链表保存元素的插入顺序, 在通过 Iterator 遍历 LinkedHashMap 时, 会按照元素的插入顺序访问元素
