---
comment: false
aside:
  toc: true

title: Java 对象引用
date: 2021-03-21 20:49
tags: Java
---

整体架构

![引用结构](https://cloudland.github.io/assets/images/202103/reference-01.png){:.rounded}

## 强引用(默认支持)

`JVM`垃圾回收对于强引用的对象, 就算出现`OOM`也不会对该对象回收。

把一个对象赋给一个引用变量, 这个引用变量就是强引用。只要还有强引用指向一个对象, 它就处于可达状态, 就表明该对象还`活着`, 垃圾收集器就不会回收该对象。即使该对象永远不会被用到`JVM`也不会回收。

```java
// 定义强引用对象
Object origin = new Object();
// target引用赋值
Object target = origin;
// 引用对象置空
origin = null;

// target 引用的 origin = new Object() 不会被GC
System.out.println(target);
```

## 软引用

软引用是相对强引用弱化一些的引用, 需要用`java.lang.ref.SoftReference`类来实现。

* 当系统内存充足时, 不回收

* 当系统内存不足时, 会回收

软引用一般用于对内存敏感的程序中。 例如高速缓存, 内存足够就保留, 不够就回收。

```java
// 构建强引用对象
Object origin = new Object();
// 构建软引用对象
SoftReference<Object> softReference = new SoftReference<>(origin);

// 手动置空强引用
origin = null;

// softReference对象会在内存不足时 GC 回收掉
```

## 弱引用

弱引用需要用`java.lang.ref.WeakReference`类来实现。只要`GC`, 无论`JVM`内存空间是否足够, 一律回收。

```java
// 构建强引用对象
Object origin = new Object();
// 构建软引用对象
WeakReference<Object> weakReference = new WeakReference<>(origin);

// 手动置空强引用
origin = null;
System.gc();

// 输入均为 null
System.out.println(origin);
System.out.println(weakReference);
```

## 虚引用

虚引用需要用`java.lang.ref.PhantomReference`类来实现。虚引用并不会决定对象的生命周期。它不能单独使用也不能通过它访问引用对象, 必须和引用队列`ReferenceQueue`联合使用。

虚引用的主要用途是跟踪对象被垃圾回收后, 会收到通知用于做一些后续处理。`Java`技术允许使用`finalize()`方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。

```java
// 定义引用对象
Object origin = new Object();
// 定义引用队列
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
// 构建虚引用
PhantomReference<Object> phantomReference = new PhantomReference<>(origin, referenceQueue);

System.gc();

// 输入 null
System.out.println(phantomReference);
// 输出引用对象
System.out.println(referenceQueue.poll());
```