---
comment: false
aside:
  toc: true

title: Seata 安装手册
date: 2021-06-03 16:19
tags: Java Alibaba Seata
---

### 1. 下载 Seata

  官网 http://seata.io/ 下载, 演示版本用的是 1.4.2(2021-04-26) 版本。

  * source 源码包, 一般用于代码研究

  * binary 编译包, 基于源码包编译后的结果。可以直接运行。

### 2. Seata 安装

  ```linux
  unzip seata-server-1.4.2.zip
  ```
### 3. Seata 配置

  可以在 seata-server-1.4.2/conf 目录， 看下配置说明`README-zh.md`清楚了解配置。如下:

  ```md
  # 脚本说明

  ## [client](https://github.com/seata/seata/tree/develop/script/client) 

  > 存放用于客户端的配置和SQL

  - at: AT模式下的 `undo_log` 建表语句
  - conf: 客户端的配置文件
  - saga: SAGA 模式下所需表的建表语句
  - spring: SpringBoot 应用支持的配置文件

  ## [server](https://github.com/seata/seata/tree/develop/script/server)

  > 存放server侧所需SQL和部署脚本

  - db: server 侧的保存模式为 `db` 时所需表的建表语句
  - docker-compose: server 侧通过 docker-compose 部署的脚本
  - helm: server 侧通过 Helm 部署的脚本
  - kubernetes: server 侧通过 Kubernetes 部署的脚本

  ## [config-center](https://github.com/seata/seata/tree/develop/script/config-center)

  > 用于存放各种配置中心的初始化脚本，执行时都会读取 `config.txt`配置文件，并写入配置中心

  - nacos: 用于向 Nacos 中添加配置
  - zk: 用于向 Zookeeper 中添加配置，脚本依赖 Zookeeper 的相关脚本，需要手动下载；ZooKeeper相关的配置可以写在 `zk-params.txt` 中
  ，也可以在执行的时候输入
  - apollo: 向 Apollo 中添加配置，Apollo 的地址端口等可以写在 `apollo-params.txt`，也可以在执行的时候输入
  - etcd3: 用于向 Etcd3 中添加配置
  - consul: 用于向 consul 中添加配置
  ```
#### 数据库配置(file.conf)

> 配置 seata-server 使用的数据源

![配置文件截图](https://cloudland.github.io/assets/images/202106/seata-2.png){:.rounded}

* 选择`mode`存储类型, 推荐选择`db`类型

* 配置 `db` 节点, 填写相关数据库连接信息

#### 注册及配置中心(registry.conf)

> 配置 seata-server 使用的注册及配置中心

* `registry`配置

  ![registry注册中心](https://cloudland.github.io/assets/images/202106/seata-3.png){:.rounded}

  * 修改`registry`配置, 推荐使用`nacos`。修改`type=nacos`

  * 基于配置`type=nacos`为注册中心, 修改`nacos`配置节点

* `config`配置

  ![registry配置中心](https://cloudland.github.io/assets/images/202106/seata-4.png){:.rounded}

  * 修改`config`配置, 推荐使用`nacos`。修改`type=nacos`

  * 基于配置`type=nacos`为配置中心, 修改`nacos`配置节点

### 4. Seata 配置中心

具体可参照 -> [Seata参数配置](https://seata.io/zh-cn/docs/user/configurations.html)

项目采用的是Nacos作为配置中心, 需要执行[config-center](https://github.com/seata/seata/tree/develop/script/config-center)的`nacos`目录下`nacos-config.sh`脚本, 将配置信息初始化到`nacos`上。

```shell
sh nacos-config.sh -h 127.0.0.1
```

* -h: 主机

* -p: 端口

* -g: 分组

* -t: 命名空间

**注:** [config.txt](https://cloudland.github.io/assets/images/202106/01-seata/config.txt) 及 [nacos-config.sh](https://cloudland.github.io/assets/images/202106/01-seata/nacos-config.sh)

### 5. Seata 数据库

#### TC 事物控制者

> 事物控制者, 由`seata-server`负责协调控制。这块内容只有在`file.conf`中配置`type=db`, 方需要做数据库表的初始化工作。

* PostgreSQL

  ```sql
  -- -------------------------------- The script used when storeMode is 'db' --------------------------------
  -- the table to store GlobalSession data
  CREATE TABLE IF NOT EXISTS public.global_table
  (
      xid                       VARCHAR(128) NOT NULL,
      transaction_id            BIGINT,
      status                    SMALLINT     NOT NULL,
      application_id            VARCHAR(32),
      transaction_service_group VARCHAR(32),
      transaction_name          VARCHAR(128),
      timeout                   INT,
      begin_time                BIGINT,
      application_data          VARCHAR(2000),
      gmt_create                TIMESTAMP(0),
      gmt_modified              TIMESTAMP(0),
      CONSTRAINT pk_global_table PRIMARY KEY (xid)
  );

  CREATE INDEX idx_gmt_modified_status ON public.global_table (gmt_modified, status);
  CREATE INDEX idx_transaction_id ON public.global_table (transaction_id);

  -- the table to store BranchSession data
  CREATE TABLE IF NOT EXISTS public.branch_table
  (
      branch_id         BIGINT       NOT NULL,
      xid               VARCHAR(128) NOT NULL,
      transaction_id    BIGINT,
      resource_group_id VARCHAR(32),
      resource_id       VARCHAR(256),
      branch_type       VARCHAR(8),
      status            SMALLINT,
      client_id         VARCHAR(64),
      application_data  VARCHAR(2000),
      gmt_create        TIMESTAMP(6),
      gmt_modified      TIMESTAMP(6),
      CONSTRAINT pk_branch_table PRIMARY KEY (branch_id)
  );

  CREATE INDEX idx_xid ON public.branch_table (xid);

  -- the table to store lock data
  CREATE TABLE IF NOT EXISTS public.lock_table
  (
      row_key        VARCHAR(128) NOT NULL,
      xid            VARCHAR(128),
      transaction_id BIGINT,
      branch_id      BIGINT       NOT NULL,
      resource_id    VARCHAR(256),
      table_name     VARCHAR(32),
      pk             VARCHAR(36),
      gmt_create     TIMESTAMP(0),
      gmt_modified   TIMESTAMP(0),
      CONSTRAINT pk_lock_table PRIMARY KEY (row_key)
  );

  CREATE INDEX idx_branch_id ON public.lock_table (branch_id);
  ```

* MySQL

  ```sql
  -- -------------------------------- The script used when storeMode is 'db' --------------------------------
  -- the table to store GlobalSession data
  CREATE TABLE IF NOT EXISTS `global_table`
  (
      `xid`                       VARCHAR(128) NOT NULL,
      `transaction_id`            BIGINT,
      `status`                    TINYINT      NOT NULL,
      `application_id`            VARCHAR(32),
      `transaction_service_group` VARCHAR(32),
      `transaction_name`          VARCHAR(128),
      `timeout`                   INT,
      `begin_time`                BIGINT,
      `application_data`          VARCHAR(2000),
      `gmt_create`                DATETIME,
      `gmt_modified`              DATETIME,
      PRIMARY KEY (`xid`),
      KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
      KEY `idx_transaction_id` (`transaction_id`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8;

  -- the table to store BranchSession data
  CREATE TABLE IF NOT EXISTS `branch_table`
  (
      `branch_id`         BIGINT       NOT NULL,
      `xid`               VARCHAR(128) NOT NULL,
      `transaction_id`    BIGINT,
      `resource_group_id` VARCHAR(32),
      `resource_id`       VARCHAR(256),
      `branch_type`       VARCHAR(8),
      `status`            TINYINT,
      `client_id`         VARCHAR(64),
      `application_data`  VARCHAR(2000),
      `gmt_create`        DATETIME(6),
      `gmt_modified`      DATETIME(6),
      PRIMARY KEY (`branch_id`),
      KEY `idx_xid` (`xid`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8;

  -- the table to store lock data
  CREATE TABLE IF NOT EXISTS `lock_table`
  (
      `row_key`        VARCHAR(128) NOT NULL,
      `xid`            VARCHAR(128),
      `transaction_id` BIGINT,
      `branch_id`      BIGINT       NOT NULL,
      `resource_id`    VARCHAR(256),
      `table_name`     VARCHAR(32),
      `pk`             VARCHAR(36),
      `gmt_create`     DATETIME,
      `gmt_modified`   DATETIME,
      PRIMARY KEY (`row_key`),
      KEY `idx_branch_id` (`branch_id`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8;
  ```

#### RM 事物参与者

> 事物参与者在AT模式下，需要 `undo_log` 建表语句

* PostgreSQL

  ```sql
  -- for AT mode you must to init this sql for you business database. the seata server not need it.
  CREATE TABLE IF NOT EXISTS public.undo_log
  (
      id            SERIAL       NOT NULL,
      branch_id     BIGINT       NOT NULL,
      xid           VARCHAR(128) NOT NULL,
      context       VARCHAR(128) NOT NULL,
      rollback_info BYTEA        NOT NULL,
      log_status    INT          NOT NULL,
      log_created   TIMESTAMP(0) NOT NULL,
      log_modified  TIMESTAMP(0) NOT NULL,
      CONSTRAINT pk_undo_log PRIMARY KEY (id),
      CONSTRAINT ux_undo_log UNIQUE (xid, branch_id)
  );

  CREATE SEQUENCE IF NOT EXISTS undo_log_id_seq INCREMENT BY 1 MINVALUE 1 ;
  ```

* MySQL

  ```sql
  -- for AT mode you must to init this sql for you business database. the seata server not need it.
  CREATE TABLE IF NOT EXISTS `undo_log`
  (
      `branch_id`     BIGINT       NOT NULL COMMENT 'branch transaction id',
      `xid`           VARCHAR(128) NOT NULL COMMENT 'global transaction id',
      `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
      `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
      `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
      `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
      `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
      UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
  ) ENGINE = InnoDB
    AUTO_INCREMENT = 1
    DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
  ```

### 6. Seata 启动

启动命令参数说明:

```shell
Usage: sh seata-server.sh(for linux and mac) or cmd seata-server.bat(for windows) [options]
  Options:
    --host, -h
      The host to bind.
      Default: 0.0.0.0
    --port, -p
      The port to listen.
      Default: 8091
    --storeMode, -m
      log store mode : file、db
      Default: file
    --help
```

启动命令样例:

```shell
sh seata-server.sh -p 8091 -h 127.0.0.1 -m db

# 后台运行 重定向日志文件 2>&1 是一个整体，2 表示stderr标准错误，报错内容
nohup sh seata-server.sh -p 8091 -h 127.0.0.1 -m db file.log 2>&1 &
```

#### 问题

* 问题一

  ![问题一](https://cloudland.github.io/assets/images/202106/seata-5.png){:.rounded}

  执行如下命令, 删除多余制表符

  ```shell
  sed -i 's/\r//g' nacos-config.sh
  ```

* 问题二

  ![问题二](https://cloudland.github.io/assets/images/202106/seata-6.png){:.rounded}

  是由于注册中心未做配置初始化造成。