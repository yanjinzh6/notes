---
title: 使用 Arrays 转换 List 出现不支持操作异常
date: 2020-10-24 11:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '集合']
permalink: arrays-to-list-uoe
---

## 简介

Arrays 类提供很多方法方便转换列表

一次使用 `asList` 将数组转换为列表的操作如下

```java
List<String> list = Arrays.asList(strs);
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
  String next = iterator.next();
  if ("yes".equals(next)) {
    iterator.remove();
  }
}
```

执行后抛出 `java.lang.UnsupportedOperationException` 异常

<!-- more -->

## 分析

`asList` 方法返回一个 List 对象, 这里返回的是一个内部类 `Arrays.ArrayList`, 并非常用的 `ArrayList`, 而是一个 `AbstractList`

```java
// java.util.Arrays#asList
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}

// java.util.Arrays.ArrayList
private static class ArrayList<E> extends AbstractList<E>
    implements RandomAccess, java.io.Serializable
{
    private static final long serialVersionUID = -2764017481108945198L;
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    @Override
    public int size() {
        return a.length;
    }

    @Override
    public Object[] toArray() {
        return a.clone();
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        int size = size();
        if (a.length < size)
            return Arrays.copyOf(this.a, size,
                                  (Class<? extends T[]>) a.getClass());
        System.arraycopy(this.a, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @Override
    public E get(int index) {
        return a[index];
    }

    @Override
    public E set(int index, E element) {
        E oldValue = a[index];
        a[index] = element;
        return oldValue;
    }

    @Override
    public int indexOf(Object o) {
        E[] a = this.a;
        if (o == null) {
            for (int i = 0; i < a.length; i++)
                if (a[i] == null)
                    return i;
        } else {
            for (int i = 0; i < a.length; i++)
                if (o.equals(a[i]))
                    return i;
        }
        return -1;
    }

    @Override
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(a, Spliterator.ORDERED);
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        for (E e : a) {
            action.accept(e);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        E[] a = this.a;
        for (int i = 0; i < a.length; i++) {
            a[i] = operator.apply(a[i]);
        }
    }

    @Override
    public void sort(Comparator<? super E> c) {
        Arrays.sort(a, c);
    }
}
```

`AbstractList` 中返回了默认实现的 `Iterator`

```java
// java.util.AbstractList#iterator
public Iterator<E> iterator() {
    return new Itr();
}

// java.util.AbstractList.Itr
private class Itr implements Iterator<E> {
    /**
      * Index of element to be returned by subsequent call to next.
      */
    int cursor = 0;

    /**
      * Index of element returned by most recent call to next or
      * previous.  Reset to -1 if this element is deleted by a call
      * to remove.
      */
    int lastRet = -1;

    /**
      * The modCount value that the iterator believes that the backing
      * List should have.  If this expectation is violated, the iterator
      * has detected concurrent modification.
      */
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size();
    }

    public E next() {
        checkForComodification();
        try {
            int i = cursor;
            E next = get(i);
            lastRet = i;
            cursor = i + 1;
            return next;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            AbstractList.this.remove(lastRet);
            if (lastRet < cursor)
                cursor--;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException e) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

可以看到 `java.util.AbstractList.Itr#remove` 方法中调用的是 `AbstractList.this.remove(lastRet);`

因为返回的是 `AbstractList`, 所以直接跳转到 `remove` 方法

```java
// java.util.AbstractList#remove
public E remove(int index) {
    throw new UnsupportedOperationException();
}
```

所以上面的调用会返回如下错误

```sh
ava.lang.UnsupportedOperationException: null
  at java.util.AbstractList.remove(AbstractList.java:161)
  at java.util.AbstractList$Itr.remove(AbstractList.java:374)
```

## 解决

类似这种工具类返回超类的接口, 简单的解决方法就是直接用具体实现类来处理就好了

```java
List<String> list = Arrays.asList(strs);
list = new ArrayList<String>(list);
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
  String next = iterator.next();
  if ("yes".equals(next)) {
    iterator.remove();
  }
}
```

其他方法类似
