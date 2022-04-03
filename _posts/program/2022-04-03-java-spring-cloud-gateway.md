---
comment: false
aside:
  toc: true

title: Spring Cloud Gateway 使用手册
date: 2022-04-03 13:13
tags: Java Spring SpringCloud
---

## 简介

![SpringCloudGateway架构图](https://cloudland.github.io/assets/images/program/spring/cloud/gateway/01.png){:.rounded}

## Route 路由

网关就是由多个路由规则组成，完成请求内容的转发工作。

### 简单理解

路由(Route) = 名字(id) + 映射地址(uri) + 谓词集合(Predicate) + 过滤集合(Filter)

`谓词`是用于判断请求具体走哪个`路由`. `谓词`可通过**路径**、**方法**、**请求头**等多种方式匹配。在匹配具体的`路由`后, 采用`路由`内配置的`过滤`方法, 对请求`前`或`后`做增量操作.

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
```

配置文件内容说明:

上例文件配置, 会将`http:/host:port/user/information`转发至`http://micro-service/information`服务, 并返回结果。

1. 请求被`Gateway`接收后, 通过谓词名称`Path`对应的操作类`PathRoutePredicateFactory`, 匹配配置参数`/user/**`; 

2. 匹配成功后, 获取当前`Route`转发地址`uri`: lb://micro-service;

3. 转发请求`http://micro-service/user/information`;

4. 过滤器名称`StripPrefix`对应操作类`StripPrefixGatewayFilterFactory`, 将转发的地址第一节`/user`删除，最终请求`http://micro-service/information`

#### predicates

标准配置

```yaml
spring:
    cloud:
        gateway:
            routes:
                # 路由命名，必须唯一
                - id: routeName
                  # 谓词配置
                  predicates:
                    # 谓词名称
                    - name: Path
                    # 谓词参数(参照 PathRoutePredicateFactory.Config 配置类理解)
                      args:
                        patterns: /user/**, /order/**
                        matchOptionalTrailingSeparator: true
                  uri: lb://micro-service
```

简化配置

```yaml
spring:
    cloud:
        gateway:
            routes:
                # 路由命名，必须唯一
                - id: routeName
                  # 谓词配置
                  predicates:
                    # 谓词名称=参数, 参照 PathRoutePredicateFactory#shortcutFieldOrder 理解
                    # '='号右边参数位置用','分割, 与shortcutFieldOrder函数返回参数名一一对应
                    - Path=/user/**
                  uri: lb://micro-service
```

#### filters

标准配置

```yaml
spring:
    cloud:
        gateway:
            routes:
                # 路由命名，必须唯一
                - id: routeName
                  # 过滤器配置
                  filters:
                    # 过滤器名称
                    - name: StripPrefix
                    # 过滤器参数(参照 StripPrefixGatewayFilterFactory.Config 配置类理解)
                      args:
                        parts: 1
                  uri: lb://micro-service
```

简化配置

```yaml
spring:
    cloud:
        gateway:
            routes:
                # 路由命名，必须唯一
                - id: routeName
                  filters:
                    # 路由名称=参数, 参照 StripPrefixGatewayFilterFactory#shortcutFieldOrder 理解
                    # '='号右边参数位置用','分割, 与shortcutFieldOrder函数返回参数名一一对应
                    - StripPrefix=1
                  uri: lb://micro-service
```

### 配置对应Java类

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

## Predicate 谓词

### 内置谓词 

Predicate 都是 GatewayPredicate 的实例, 主要负责请求的转发规则。

* 默认提供的谓词实现类:

```
AbstractRoutePredicateFactory
AfterRoutePredicateFactory
BeforeRoutePredicateFactory
BetweenRoutePredicateFactory
CloudFoundryRouteServiceRoutePredicateFactory
CookieRoutePredicateFactory
HeaderRoutePredicateFactory
HostRoutePredicateFactory
MethodRoutePredicateFactory
PathRoutePredicateFactory
QueryRoutePredicateFactory
ReadBodyPredicateFactory
ReadBodyRoutePredicateFactory
RemoteAddrRoutePredicateFactory
WeightRoutePredicateFactory
```

### 自定谓词

可基于项目特殊要求, 自定验证规则.

#### 自定谓词类编写

通过继承`AbstractRoutePredicateFactory`类, 并定义`Config`配置类, 重写`apply`函数便完成了自定谓词的定义。

如下样例代码:

```java
/**
 * 自定谓词
 *
 * @author 未小雨
 * @date 2022/4/3 20:32
 */
public class CustomRoutePredicateFactory extends AbstractRoutePredicateFactory<CustomRoutePredicateFactory.Config> {

    /**
     * 构造函数
     * 指定配置类类型
     */
    public CustomRoutePredicateFactory() {
        super(Config.class);
    }

    /**
     * 谓词简化配置函数
     * @return Config 类字段名集合
     */
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("field1", "field2");
    }

    /**
     * 谓词调用校验函数
     *
     * @param config 配置信息
     * @return
     */
    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        // 可使用 lambda 表达式替换 return (GatewayPredicate) serverWebExchange -> false;
        return new GatewayPredicate() {
            @Override
            public boolean test(ServerWebExchange serverWebExchange) {
                return false;
            }
        };
    }

    /**
     * 谓词配置
     */
    public static class Config {

        /**
         * 字段1
         */
        private String field1;

        /**
         * 字段2
         */
        private String field2;

        public String getField1() {
            return field1;
        }

        public void setField1(String field1) {
            this.field1 = field1;
        }

        public String getField2() {
            return field2;
        }

        public void setField2(String field2) {
            this.field2 = field2;
        }
    }

}
```

#### 配置自动注入

自定谓词编写完后, 如果不注册到容器中, 也是不会有任何效果的。我们可以通过在自定谓词类上添加`@Component`, 或使用自动注入配置类的方式`Spring`容器中。 

我们这里使用自动配置类注入的方式

```java
@Configuration(proxyBeanMethods = false)
public class ExpandAutoConfiguration {

    @Bean
	@ConditionalOnEnabledPredicate
	public CustomRoutePredicateFactory customRoutePredicateFactory() {
		return new CustomRoutePredicateFactory();
	}

}
```

#### 配置 application.yaml

```yaml
spring:
    cloud:
        gateway:
            routes:
                # 路由命名，必须唯一
                - id: custom
                # 配置谓词集合
                  predicates:
                    # 谓词名称=参数
                    - name: Custom
                      args:
                        field1: 字段值1
                        field2: 字段值2
                  uri: lb://micro-service
```

## Filter 过滤器

### 内置过滤器

Filter 都是 GatewayFilter 的实例，Filter 负责在代理服务`之前`或`之后`做一些事情。

* 默认提供的过滤实现类:

```
AddRequestHeaderGatewayFilterFactory
AddRequestParameterGatewayFilterFactory
AddResponseHeaderGatewayFilterFactory
DedupeResponseHeaderGatewayFilterFactory
HystrixGatewayFilterFactory
MapRequestHeaderGatewayFilterFactory
PrefixPathGatewayFilterFactory
PreserveHostHeaderGatewayFilterFactory
RedirectToGatewayFilterFactory
RemoveRequestHeaderGatewayFilterFactory
RemoveRequestParameterGatewayFilterFactory
RemoveResponseHeaderGatewayFilterFactory
RequestHeaderSizeGatewayFilterFactory
RequestHeaderToRequestUriGatewayFilterFactory
RequestSizeGatewayFilterFactory
RetryGatewayFilterFactory
RewriteLocationResponseHeaderGatewayFilterFactory
RewritePathGatewayFilterFactory
RewriteResponseHeaderGatewayFilterFactory
SaveSessionGatewayFilterFactory
SecureHeadersGatewayFilterFactory
SetPathGatewayFilterFactory
SetRequestHeaderGatewayFilterFactory
SetRequestHostHeaderGatewayFilterFactory
SetResponseHeaderGatewayFilterFactory
SetStatusGatewayFilterFactory
SpringCloudCircuitBreakerFilterFactory
StripPrefixGatewayFilterFactory
```

### 自定过滤器

#### 自定过滤器类编写

```java
/**
 * 字段过滤器
 *
 * @author 未小雨
 * @date 2022/4/3 21:19
 */
public class CustomGatewayFilterFactory extends AbstractGatewayFilterFactory<CustomGatewayFilterFactory.Config> {

    /**
     * 构造函数
     * 指定配置类类型
     */
    public CustomGatewayFilterFactory() {
        super(Config.class);
    }

    /**
     * 过滤器简化配置函数
     * @return Config 类字段名集合
     */
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("field");
    }

    @Override
    public GatewayFilter apply(Config config) {
        // 可使用 lambda 表达式替换 return (exchange, chain) -> null;
        return new GatewayFilter() {
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                // chain.filter(exchange.mutate().request(newRequest).build())
                return null;
            }
        };
    }

    /**
     * 过滤器配置
     */
    public static class Config {

        /**
         * 字段1
         */
        private String field;

        public String getField() {
            return field;
        }

        public void setField(String field) {
            this.field = field;
        }
    }

}
```

#### 配置自动注入

```java
@Configuration(proxyBeanMethods = false)
public class ExpandAutoConfiguration {

    @Bean
	@ConditionalOnEnabledPredicate
	public CustomGatewayFilterFactory customGatewayFilterFactory() {
		return new CustomGatewayFilterFactory();
	}

}
```

#### 配置 application.yaml

```yaml
spring:
    cloud:
        gateway:
            routes:
                # 路由命名，必须唯一
                - id: routeName
                  # 过滤器配置
                  filters:
                    # 过滤器名称
                    - name: Custom
                    # 过滤器参数(参照 StripPrefixGatewayFilterFactory.Config 配置类理解)
                      args:
                        field1: 字段值1
                  uri: lb://micro-service
```

## SpringGateway自动装载

### GatewayAutoConfiguration


* GatewayProperties

    用来初始配置类`RouteDefinition(路由信息)`、`FilterDefinition(过滤器信息)`、`PredicationDefinition(断言信息)`字段数据

    ```java
    @Bean
	@ConditionalOnMissingBean
	public PropertiesRouteDefinitionLocator propertiesRouteDefinitionLocator(GatewayProperties properties) {
		return new PropertiesRouteDefinitionLocator(properties);
	}
    ```

* RouteDefinitionLocator

    存储从配置文件中读取的路由信息

    ```java
    @Bean
	@Primary
	public RouteDefinitionLocator routeDefinitionLocator(List<RouteDefinitionLocator> outeDefinitionLocators){
		return new CompositeRouteDefinitionLocator(
				Flux.fromIterable(routeDefinitionLocators));
	}
    ```

* RouteLocator `重要`

    初始整个路由配置信息类

    ```java
    @Bean
	public RouteLocator routeDefinitionRouteLocator(GatewayProperties properties,
			List<GatewayFilterFactory> gatewayFilters,
			List<RoutePredicateFactory> predicates,
			RouteDefinitionLocator routeDefinitionLocator,
			ConfigurationService configurationService) {
		return new RouteDefinitionRouteLocator(routeDefinitionLocator, predicates,
				gatewayFilters, properties, configurationService);
	}
    ```

### RouteDefinitionRouteLocator `重要`

此类主要是组装了`RouteDefinitionLocator`、`RoutePredicateFactory`、`GatewayFilterFactory`, 从而通过这些信息转换成`Route`.


* 基于配置初始谓词

```java
// 初始 Predicate 函数
private void initFactories(List<RoutePredicateFactory> predicates) {
	predicates.forEach(factory -> {
        // key = RoutePredicateFactory 实现类的名称前缀，如: PathRoutePredicateFactory 取前缀 key = Path
		String key = factory.name();
		if (this.predicates.containsKey(key)) {
			this.logger.warn("A RoutePredicateFactory named " + key
					+ " already exists, class: " + this.predicates.get(key)
					+ ". It will be overwritten.");
		}
        // 已存在该断言 Factory, 覆盖原有值。 以SCG内置的提供为主。尽量避免重名。
		this.predicates.put(key, factory);
		if (logger.isInfoEnabled()) {
			logger.info("Loaded RoutePredicateFactory [" + key + "]");
		}
	});
}

// 谓词执行
private AsyncPredicate<ServerWebExchange> lookup(RouteDefinition route, PredicateDefinition predicate) {
    // 查找 RoutePredicateFactory
    RoutePredicateFactory<Object> factory = this.predicates.get(predicate.getName());
	if (factory == null) {
		throw new IllegalArgumentException(
				"Unable to find RoutePredicateFactory with name "
						+ predicate.getName());
	}
	if (logger.isDebugEnabled()) {
		logger.debug("RouteDefinition " + route.getId() + " applying "
				+ predicate.getArgs() + " to " + predicate.getName());
	}
        
    /**
     * 重点: 
     * 此处将 PredicateDefinition 配置的数据, 转换成 Config 对象, 再调用 RoutePredicateFactory 接口的 applyAsync 函数.
     * 函数内部会调用 RoutePredicateFactory 具体实现的类的 apply 函数, 这样实现类就会传入 config 配置信息
     */        
	// @formatter:off
	Object config = this.configurationService.with(factory)
			.name(predicate.getName())
			.properties(predicate.getArgs())
			.eventFunction((bound, properties) -> new PredicateArgsEvent(
					RouteDefinitionRouteLocator.this, route.getId(), properties))
			.bind();
	// @formatter:on

    // 调用谓词
	return factory.applyAsync(config);
}
```

* 基于配置初始过滤

```java
// 初始 Filter 函数
private List<GatewayFilter> getFilters(RouteDefinition routeDefinition) {
	List<GatewayFilter> filters = new ArrayList<>();

    // 添加内置过滤器
	if (!this.gatewayProperties.getDefaultFilters().isEmpty()) {
		filters.addAll(loadGatewayFilters(DEFAULT_FILTERS,
				new ArrayList<>(this.gatewayProperties.getDefaultFilters())));
	}

    // 添加配置过滤器
	if (!routeDefinition.getFilters().isEmpty()) {
		filters.addAll(loadGatewayFilters(routeDefinition.getId(),
				new ArrayList<>(routeDefinition.getFilters())));
	}

    // 排序
	AnnotationAwareOrderComparator.sort(filters);
	return filters;
}

// 过滤器执行
@SuppressWarnings("unchecked")
List<GatewayFilter> loadGatewayFilters(String id, List<FilterDefinition> filterDefinitions) {
	ArrayList<GatewayFilter> ordered = new ArrayList<>(filterDefinitions.size());
	for (int i = 0; i < filterDefinitions.size(); i++) {
		FilterDefinition definition = filterDefinitions.get(i);
		GatewayFilterFactory factory = this.gatewayFilterFactories.get(definition.getName());
		if (factory == null) {
			throw new IllegalArgumentException(
					"Unable to find GatewayFilterFactory with name "
								+ definition.getName());
		}
		if (logger.isDebugEnabled()) {
			logger.debug("RouteDefinition " + id + " applying filter "
					+ definition.getArgs() + " to " + definition.getName());
		}

        /**
        * 重要:
        * 获取配置文件配置参数
        */
		// @formatter:off
		Object configuration = this.configurationService.with(factory)
				.name(definition.getName())
				.properties(definition.getArgs())
				.eventFunction((bound, properties) -> new FilterArgsEvent(
						// TODO: why explicit cast needed or java compile fails
						RouteDefinitionRouteLocator.this, id, (Map<String, Object>) properties))
				.bind();
		// @formatter:on
		// some filters require routeId
		// TODO: is there a better place to apply this?
		if (configuration instanceof HasRouteId) {
			HasRouteId hasRouteId = (HasRouteId) configuration;
			hasRouteId.setRouteId(id);
		}
		
        // 调用过滤器
        GatewayFilter gatewayFilter = factory.apply(configuration);
		if (gatewayFilter instanceof Ordered) {
			ordered.add(gatewayFilter);
		}
		else {
			ordered.add(new OrderedGatewayFilter(gatewayFilter, i + 1));
		}
	}

	return ordered;
}
```

### 
