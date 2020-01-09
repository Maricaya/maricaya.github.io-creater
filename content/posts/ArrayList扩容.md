---
title: "初识Collection体系-ArrayList的扩容机制jdk8"
date: 2019-12-16T17:00:30+08:00
draft: false
---

请写一篇博客，总结一下你在Collection体系中学到的知识，例如可以是：

Collection体系的常用类及其背后的数据结构
Collection体系的常用类及其背后的数据结构对比
ArrayList源码阅读
ArrayList是如何扩容的？
HashMap源码阅读
HashMap是如何扩容的？
HashMap从Java7到Java8发生了哪些变化？
为什么HashMap不是线程安全的？
----------------------------------
想了解ArrayList的动态扩容机制，最好的方式是阅读源码。

## ArrayList的构造方法 - 有初始容量
初始容量 > 0时，elementData初始化为initialCapacity大小的数组。
```java
/**
  * 构造具有指定初始容量的空列表 list。
  *
  * @param  initialCapacity  list的初始容量
  * @throws IllegalArgumentException 如果初始容量为负
  */
public ArrayList(int initialCapacity) {
  if (initialCapacity > 0) {
    this.elementData = new Object[initialCapacity];
  } else if (initialCapacity == 0) {
    this.elementData = EMPTY_ELEMENTDATA;
  } else {
    throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
  }
  }
}
```

## ArrayList的扩容机制 - add()内部实现
### add方法
```java
/**
  * 将 指定元素 加入list末尾
  * @param e  将被加入list的元素
  * @return <tt>true</tt> (as specified by {@link Collection#add})
  */
public boolean add(E e) {
  // 确保有足够的空间
  // 在添加元素前进行判断，如果空间不够就扩容
  ensureCapacityInternal(size + 1);  // Increments modCount!! 增加模式计数
  elementData[size++] = e;
  return true;
}
```
### ensureCapacityInternal方法
```java
private void ensureCapacityInternal(int minCapacity) {
  // 保证有足够的容量     // 计算容量
  this.ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
// 计算容量方法
private static int calculateCapacity(Object[] elementData, int minCapacity) {
  // 这里就是判断elementData（初始数组）是不是为空数组
  // 如果是，容量取 默认容量（10）和 size+1(第一次调用add传入的minCapacity) 的最大值
  // 如果不是，取 size+1
  if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    return Math.max(DEFAULT_CAPACITY, minCapacity);
  }
  return minCapacity;
}
```
## ensureExplicitCapacity方法
```java
private void ensureExplicitCapacity(int minCapacity) {
  modCount++;
  // 溢出
  if (minCapacity - elementData.length > 0) {
    grow(minCapacity);
  }
}
```

## grow方法 - 扩容的主要方式
grow 中的主要操作是,新建了一个新数组，容量扩大为1.5倍，然后将原来的数组都拷贝进去。
```java
 /**
  * 增加容量，至少保证能够容纳 minCapacity 指定的最小容量
  *
  * @param minCapacity 需要的最小容量
  */
private void grow(int minCapacity) {
    // overflow-conscious code
    // 旧数组的容量
    int oldCapacity = elementData.length;
    // 为新数组的容量 （旧容量的1.5倍）
    int newCapacity = oldCapacity + (oldCapacity >> 1);
     // 1.5倍容量 < 需要的最小容量； newCapacity = 最小容量
    if (newCapacity - minCapacity < 0) { newCapacity = minCapacity; }
    // 1.5倍容量 > 数组最大长度，使用hugeCapacity比较二者
    if (newCapacity - MAX_ARRAY_SIZE > 0) { newCapacity = hugeCapacity(minCapacity); }
    // minCapacity is usually close to size, so this is a win:
    // 将原数组中的元素拷贝进新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## hugeCapacity方法

```java
private static int hugeCapacity(int minCapacity) {
  // overflow 溢出，报错
  if (minCapacity < 0) { 
    throw new OutOfMemoryError();
  }
  // 比较 minCapacity MAX_ARRAY_SIZE， 将最大的那个作为新数组大小
  // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
  return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

## 完整走一遍 空数组add(1)为例
```java
List a = new ArrayList<>(0);
a.add(1);
```
1. add(1) 将第一个元素1添加到list中，调用 `ensureCapacityInternal(0+1);`
```java
public boolean add(E e) {
  ensureCapacityInternal(size + 1); // ensureCapacityInternal(0 + 1)
  // xxx
}
```
2. `ensureCapacityInternal(0+1)`
先执行`calculateCapacity([], 0+1)`
再执行`ensureExplicitCapacity(10)`
```java
private void ensureCapacityInternal(int minCapacity) {
  // 
  this.ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
  if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    // 是空数组
    // Math.max(10, 0+1) ： 10
    return Math.max(DEFAULT_CAPACITY, minCapacity);
  }
  return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
  modCount++;
  // 溢出
  if (minCapacity - elementData.length > 0) {
    // 10 - 0 > 0 执行grow(10)
    grow(minCapacity);
  }
}
```
3. `grow(10)`
```java
private void grow(int minCapacity) {
  // oldCapacity: 0
  int oldCapacity = elementData.length;
  // newCapacity: 0
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  // 0 - 10 < 0 : newCapacity = 10
  if (newCapacity - minCapacity < 0) { newCapacity = minCapacity; }
  // 10 - (Integer.MAX_VALUE - 8) < 0 不执行
  if (newCapacity - MAX_ARRAY_SIZE > 0) { newCapacity = hugeCapacity(minCapacity); }
  // 将原数组中的元素拷贝进新数组
  elementData = Arrays.copyOf(elementData, newCapacity);
}
```
