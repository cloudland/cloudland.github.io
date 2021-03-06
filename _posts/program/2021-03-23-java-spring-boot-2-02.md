---
comment: false
aside:
  toc: true

title: SpringBoot2 自动配置
date: 2021-03-23 17:00
tags: Java Spring SpringBoot
---

## 解析自动配置

**@EnableAutoConfiguration 开启自动配置**

![EnableAutoConfiguration](https://cloudland.github.io/assets/images/202103/springboot-20.png){:.rounded}

### @AutoConfigurationPackage

自动配置包

![AutoConfigurationPackage](https://cloudland.github.io/assets/images/202103/springboot-25.png){:.rounded}{:height="60%" width="60%"}

![AutoConfigurationPackage](https://cloudland.github.io/assets/images/202103/springboot-21.png){:.rounded}

`new PackageImports(metadata).getPackageNames()`{:.info}获取注解应用的类所在包路径。

**例如:**

注解在`org.lei.server.StudyApplication`类, 将获取`org.lei.server`路径。利用`register`导入一系列组件。

### @Import(AutoConfigurationImportSelector.class)

查看`selectImports`{:.info}函数

![AutoConfigurationImportSelector](https://cloudland.github.io/assets/images/202103/springboot-22.png){:.rounded}

```mermaid
graph LR;
    A[getAutoConfigurationEntry]
    B[getCandidateConfigurations]
    C[SpringFactoriesLoader.loadFactoryNames]
    D[loadSpringFactories]
    E[META-INF/spring.factories]
    A-->B;
    B-->C;
    C-->D;
    D-->E;
```

![loadSpringFactories](https://cloudland.github.io/assets/images/202103/springboot-23.png){:.rounded}

```java
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

* 获取所有存在`META-INF/spring.factories`文件的路径

* 通过`PropertiesLoaderUtils.loadProperties`加载存在的路径, 获取`org.springframework.boot.autoconfigure.EnableAutoConfiguration`自动配置类

### 条件装配

可以结合**@Conditional**最终决定是否装配

## 自定 Starter

### 项目名-spring-boot-starter(启动器)

```xml
<groupId>项目名</groupId>
<artifactId>项目名-spring-boot-starter</artifactId>
<version>版本</version>

<dependencies>
  <!-- 引入自动配置包 -->
  <dependency>
     <groupId>项目名</groupId>
     <artifactId>项目名-spring-boot-starter-autoconfigure</artifactId>
     <version>版本</version>
  </dependency>
</dependencies>
```

### 项目名-spring-boot-starter-autoconfigure(自动配置包)

* 定义模块POM

```xml
<groupId>项目名</groupId>
<artifactId>项目名-spring-boot-starter-autoconfigure</artifactId>
<version>版本</version>
```

* 定义自动配置类

```java
@Configuration
public class XxxAutoConfiguration {

}
```

* 新增`META-INF`下`spring.factories`文件

```yml
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
XxxAutoConfiguration
```