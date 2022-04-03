---
comment: false
aside:
  toc: true

title: Spring Cloud Gateway 
date: 2022-04-03 13:13
tags: Java Spring SpringCloud
---

## 简介

![SpringCloudGateway架构图](https://cloudland.github.io/assets/images/program/spring/cloud/gateway/01.png){:.rounded}

## Route 路由

网关就是由多个路由规则组成，完成请求内容的转发工作。

### 简单理解

路由(Route) = 名字(id) + 映射地址(uri) + 谓词集合(Predicate) + 过滤集合(Filter)

### 路由配置

* application.yaml 

```yaml
spring:
    cloud:
        gateway:
            discovery:
                locator:
                # 关闭 Gateway 自动转发所有微服务, 避免直接基于微服务名直接通过网关就可以访问
                    enabled: false
                # 配置路由
            routes:
                # 路由命名，必须唯一
                - id: routeName
                # 配置谓词集合
                    predicates:
                        # 谓词名称=参数
                        - Path=/user/** 
                    # 配置过滤器集合
                    filters:
                        # 路由名称=参数
                        - StripPrefix=1
                    # lb: LoadBalance 转发的微服务名称
                    uri: lb://micro-service
                    metadata:
                    order:
```

配置文件内容说明:

上例文件配置, 会将`http:/host:port/user/information`转发至`http://micro-service/information`服务, 并返回结果。

1. 请求被`Gateway`接收后, 通过谓词名称`Path`对应的操作类`PathRoutePredicateFactory`, 匹配配置参数`/user/**`; 

2. 匹配成功后, 获取当前`Route`转发地址`uri`: lb://micro-service;

3. 转发请求`http://micro-service/user/information`;

4. 过滤器名称`StripPrefix`对应操作类`StripPrefixGatewayFilterFactory`, 将转发的地址第一节`/user`删除，最终请求`http://micro-service/information`


* 配置对应Java类

```java
package org.springframework.cloud.gateway.route;

@Validated
public class RouteDefinition {

    // 名称
    private String id;

    // 谓词集合
    @NotEmpty
    @Valid
    private List<PredicateDefinition> predicates = new ArrayList<>();

    // 过滤集合
    @Valid
    private List<FilterDefinition> filters = new ArrayList<>();

    // 映射地址
    @NotNull
    private URI uri;

    // 元数据
    private Map<String, Object> metadata = new HashMap<>();

    // 路由顺序
    private int order = 0;

    ...
}
```

### Predicate 谓词

Predicate 都是 RoutePredicateFactory 的实例, 主要负责请求的转发规则。

![SpringCloudGatewayPredicate实现类](https://cloudland.github.io/assets/images/program/spring/cloud/gateway/02.png){:.rounded}


### Filter 过滤器

Filter 都是 GatewayFilter 的实例，Filter 负责在代理服务`之前`或`之后`做一些事情。

![SpringCloudGatewayPredicate实现类](https://cloudland.github.io/assets/images/program/spring/cloud/gateway/03.png){:.rounded}

