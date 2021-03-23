---
comment: false
aside:
  toc: true

title: SpringBoot2 自动配置
date: 2021-03-23 17:00
tags: Java Spring SpringBoot
---

## 

### @EnableAutoConfiguration

* AutoConfigurationPackage

![EnableAutoConfiguration](https://cloudland.github.io/assets/images/202103/springboot-20.png){:.rounded}

### @AutoConfigurationPackage

自动配置包

![EnableAutoConfiguration](https://cloudland.github.io/assets/images/202103/springboot-21.png){:.rounded}

`new PackageImports(metadata).getPackageNames()`获取注解应用的类所在包路径。例如注解在`org.lei.server.StudyApplication`类, 将获取`org.lei.server`路径。利用`register`导入一系列组件。

### @Import(AutoConfigurationImportSelector.class)

![EnableAutoConfiguration](https://cloudland.github.io/assets/images/202103/springboot-22.png){:.rounded}

* 利用`getAutoConfigurationEntry`函数给容器批量导入一些组件

```mermaid
graph TB;
    A[getAutoConfigurationEntry]
    B[getCandidateConfigurations]
    C[SpringFactoriesLoader.loadFactoryNames]
    D[loadSpringFactories]
    E[META-INF/spring.factories#org.springframework.boot.autoconfigure.EnableAutoConfiguration]
    A-->B;
    B-->C;
    C-->D;
    D-->E;
```

![loadSpringFactories](https://cloudland.github.io/assets/images/202103/springboot-23.png){:.rounded}

![FACTORIES_RESOURCE_LOCATION](https://cloudland.github.io/assets/images/202103/springboot-24.png){:.rounded}

`org.springframework.boot.autoconfigure.EnableAutoConfiguration`


* `getAutoConfigurationEntry` -> `getCandidateConfigurations` -> ``
