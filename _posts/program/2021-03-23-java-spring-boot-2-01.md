---
comment: false
aside:
  toc: true

title: SpringBoot2 基础原理
date: 2021-03-23 16:00
tags: Java Spring SpringBoot
---

## 原理解析

### 启动过程

> SpringBoot 启动分两部分: new, run

![SpringApplication](https://cloudland.github.io/assets/images/202103/springboot-10.png){:.rounded}

#### new

创建`SpringApplication`实例并初始化属性

![SpringApplication](https://cloudland.github.io/assets/images/202103/springboot-11.png){:.rounded}

`getSpringFactoriesInstances`去`META-INF`文件夹内的`spring.factories`找`1`、`2`配置类集合

1. 查找`ApplicationContextInitializer`配置并设置`<a href="#initializers">initializers</a>`属性;

2. 查找`ApplicationListener`配置并设置`listeners`属性;

#### run

![ApplicationListener](https://cloudland.github.io/assets/images/202103/springboot-12.png){:.rounded}

* `configureHeadlessProperty`设置`java.awt.headless`为`true`;

* `getRunListeners`从`META-INF`文件夹内的`spring.factories`中获取配置的`SpringApplicationRunListener`运行时监听器

    * starting:注册完直接执行

    * environmentPrepared:在`prepareEnvironment`内, 环境准备好时调用

    * contextPrepared:在`prepareContext`内, 容器上下文准备好时调用

    * contextLoaded:在`prepareContext`内, 容器上下文加载完时调用
 
    * started:容器启动完成后调用

    * failed:初始过程异常调用

* `prepareEnvironment`准备基础环境信息。读取所有的配置源配置信息

* `createApplicationContext`创建IOC容器

    根据应用类型创建容器

    * SERVLET 传统的

        org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext

    * REACTIVE 响应式

        org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext

    ![ApplicationListener](https://cloudland.github.io/assets/images/202103/springboot-13.png){:.rounded}

    
* `prepareContext`准备容器信息

    ![ApplicationListener](https://cloudland.github.io/assets/images/202103/springboot-14.png){:.rounded}

    * `applyInitializers`应用初始化器, 获取`[initializers](#initializers)`集合循环执行`initialize`函数。对容器初始化扩展

* `refreshContext`刷新容器, 整个初始化过程。(模板方法模式)

* `callRunners`回调[Runner](#runner)

* `handleRunFailure`容器初始、启动过程异常时调用


### 事件与监听

> 需要在`META-INF`文件夹内的`spring.factories`配置, Spring方能读取加载。

* ApplicationContextInitializer

> `org.springframework.context.ApplicationContextInitializer`

![ApplicationContextInitializer](https://cloudland.github.io/assets/images/202103/springboot-02.png){:.rounded}


* ApplicationListener

> `org.springframework.context.ApplicationListener`

![ApplicationListener](https://cloudland.github.io/assets/images/202103/springboot-01.png){:.rounded}


* SpringApplicationRunListener

> `org.springframework.context.SpringApplicationRunListener`

![SpringApplicationRunListener](https://cloudland.github.io/assets/images/202103/springboot-03.png){:.rounded}

### <a href="#runner">Runner</a>

合并`ApplicationRunner`和`CommandLineRunner`按`@Order`排序执行, 执行接口`run`函数。

* ApplicationRunner

![ApplicationRunner](https://cloudland.github.io/assets/images/202103/springboot-04.png){:.rounded}

* CommandLineRunner

![CommandLineRunner](https://cloudland.github.io/assets/images/202103/springboot-05.png){:.rounded}

