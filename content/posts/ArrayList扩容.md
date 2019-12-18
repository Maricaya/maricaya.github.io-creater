---
title: "初识Collection体系-ArrayList的扩容机制"
date: 2019-12-16T17:00:30+08:00
draft: true
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

## ArrayList的构造方法 - 有初始容量
初始容量 > 0时，elementData初始化为initialCapacity大小的数组。
```java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

## ArrayList的扩容机制 - add()内部实现
### add方法
```java
public boolean add(E var1) {
  // 确保有足够的空间
  // 在添加元素前进行判断，如果空间不够就扩容
  this.ensureCapacityInternal(this.size + 1);
  this.elementData[this.size++] = var1;
  return true;
}
```
### ensureCapacityInternal方法
```java
private void ensureCapacityInternal(int var1) {
  // 保证有足够的容量     // 计算容量
  this.ensureExplicitCapacity(calculateCapacity(this.elementData, var1));
}
// 计算容量方法
private static int calculateCapacity(Object[] var0, int var1) {
  // 这里就是判断elementData（初始数组）是不是为空数组
  // 如果是，容量取 默认容量（10）和 size+1(第一次调用add传入) 的最大值
  // 如果不是，取 size+1
  return var0 == DEFAULTCAPACITY_EMPTY_ELEMENTDATA ? Math.max(DEFAULT_CAPACITY, var1) : var1;
}
```
## ensureExplicitCapacity方法
```java
private void ensureExplicitCapacity(int var1) {
  ++this.modCount;
  if (var1 - this.elementData.length > 0) {
    this.grow(var1);
    // grow 中的操作是,新建了一个新数组,然后将原来的东西都拷贝进去
    // newCapacity = oldCapacity + (oldCapacity >> 1) (0.5倍)
  }
}
```
