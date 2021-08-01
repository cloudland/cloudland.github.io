---
comment: false
aside:
  toc: true

title: Seata 使用手册
date: 2021-08-01 17:20
tags: Java Alibaba Seata
---

### Maven 

* 基于项目需求导入`spring-cloud-starter-alibaba-seata`适合的版本, 这里导入的是`2.2.4.RELEASE`版本，里面包含的`io.seata:seata-all:1.3.0`；

* `druid-spring-boot-starter`阿里提供的数据库连接池框架，`Seata`事务代理会用到；

```xml
<!-- Alibaba Seata Begin -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <version>2.2.4.RELEASE</version>
</dependency>
<!-- Alibaba Seata End -->

<!-- Alibaba Druid Begin -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.0</version>
</dependency>
<!-- Alibaba Druid End -->
```

### 配置参数

可配置[参数](https://cloudland.github.io/assets/images/202106/01-seata/application.yml)，如下是需要修改的配置的参数：

```yaml
seata:
  # 是否启用
  enabled: true
  # 应用编号
  application-id: seata-business-server
  # 事务群组（可以每个应用独立取名，也可以使用相同的名字）
  tx-service-group: my_test_tx_group 
  service:
    # TC 集群（必须与seata-server保持一致）
    vgroup-mapping:
      # 这里的Key是tx-service-group配置的内容
      tx-service-group: default
    # 降级开关
    enable-degrade: false
    # 禁用全局事务（默认false）
    disable-global-transaction: false
    grouplist:
      default: 127.0.0.1:8091
  # 注册中心
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      group : SEATA_GROUP
      cluster: default
  # 配置中心
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
```

#### 重点配置

* 简要说明:

    yaml配置：

    ```markdown
    tx-service-group: **my_test_tx_group**

    service:
        vgroup-mapping:
        **tx-service-group**: *default*
    ```
    
    对应seata-server配置：

    ```markdown
    service.vgroupMapping.**my_test_tx_group**=*default*

    这就是具体的TC服务列表
    service.*default*.grouplist=127.0.0.1:8091
    ```

* 详细说明(慢慢看，不宜看懂)：

    基于样例给出的配置，说明其关系:

    框架`seata`获取配置`tx-service-group`的值为`my_test_tx_group`，在查找配置`vgroup-mapping`(Map类型)，配置的`key`为`my_test_tx_group`内容。与`Nacos`配置中心上，配置的`service.vgroupMapping.my_test_tx_group`是否一致。取出配置值`default`。拿着这个值，拼接为`	
    service.default.grouplist`再在`Nacos`查找，获取其值为`TC`的服务器列表。

    ![Seata](https://cloudland.github.io/assets/images/202106/seata-7.png)

* 可以添加多个`service.vgroupMapping.my_test_tx_group`，替换`my_test_tx_group`定义成不同的名称。与`application.yaml`文件中配置的`tx-service-group`值一致，且`vgroup-mapping`添加`key`为此定义名称，值为定义的变量`service.default.grouplist`中间`default`，此值也可以替换，要与`service.vgroupMapping.my_test_tx_group`给的值匹配上即可。


### 编码使用

#### 样例代码

> 只需在需要控制全局事务的服务上, 添加`@GlobalTransactional`即可。无需在每个被调用的子系统的服务上再添加任何注解。

```java
@Service("Web.Study.SeataMyBatisService")
public class SeataMyBatisService extends AbstractParentService {

    /**
     * 微服务(A)调用服务
     */
    @Resource
    private org.cloudland.study.remote.micro.a.MyBatisFeignClient aClient;

    /**
     * 微服务(B)调用服务
     */
    @Resource
    private org.cloudland.study.remote.micro.b.MyBatisFeignClient bClient;

    /**
     * 两个微服务
     *
     * @param seataDo
     */
    @GlobalLock
    @GlobalTransactional(rollbackFor = {Exception.class, RuntimeException.class}, timeoutMills = 300000)
    public void doTransaction(SeataTransactionDo seataDo) {
        getLogger().info("开始全局事务，XID = " + RootContext.getXID());

        // 调用微服务(A)
        org.cloudland.study.remote.micro.a.to.TransactionTestTo a = new org.cloudland.study.remote.micro.a.to.TransactionTestTo(seataDo.getId(), seataDo.getTitle(), seataDo.getContent());
        Result<Void> aResult = aClient.doTransaction(a);
        getLogger().info("微服务(A): {}", aResult.getCode());

        // 调用微服务(B)
        org.cloudland.study.remote.micro.b.to.TransactionTestTo b = new org.cloudland.study.remote.micro.b.to.TransactionTestTo(seataDo.getId(), seataDo.getTitle(), seataDo.getContent());
        Result<Void> bResult = bClient.doTransaction(b);
        getLogger().info("微服务(B): {}", bResult.getCode());

    }

}
```

#### @GlobalTransactional

* name

    全局事务名称

* timeoutMills

    超时时间。默认：60000

*  rollbackFor

    回滚异常

* noRollbackFor

    非回滚异常

### 可能遇到的问题