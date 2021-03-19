---
comment: false
aside:
  toc: true

title: 解读 AbstractQueuedSynchronizer
date: 2021-03-19 09:55
tags: Java AQS
---

## 简介

AbstractQueuedSynchronizer简称AQS。提供了一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架。

* 提供基于一个FIFO的等待队列;
* 使用原子变量`state`{:.success}维护锁状态, 并通过CAS方式更新;

