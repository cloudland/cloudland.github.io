---
comment: false
aside:
  toc: true

title: 设计模式
date: 2021-03-17 18:00
tags: 设计思想 设计模式
---

## 单例模式

### 高并发场景

双端检测机制(Double Check Lock)

```java
// 添加 volatile 禁止指令重排
private volatile PseudoSingleton instance = null;

public static PseudoSingleton getInstance() {
    // 检查是否初始化
    if (null == instance) {
        // 加锁
        synchronized(PseudoSingleton.class) {
            //检查初始化
            if (null == instance) {
                 instance = new PseudoSingleton();
            }
        }

    }

    return instance;
}
```