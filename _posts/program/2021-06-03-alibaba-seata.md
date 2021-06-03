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

  可以在 seata-server-1.4.2/conf 目录， 看下配置说明`README-zh.md`清楚了解配置。如下图:

  ![配置文件](https://cloudland.github.io/assets/images/202106/seata-1.png){:.rounded}

  配置分两大部分, 数据库和注册、配置中心

  * file.conf

    > 用于配置数据库链接相关信息

    ![配置文件截图](https://cloudland.github.io/assets/images/202106/seata-2.png){:.rounded}

    * 选择`mode`存储类型, 推荐选择`db`类型

    * 配置 `db` 节点, 填写相关数据库连接信息

  * registry.conf

    > 用于配置注册、配置中心相关信息

    ![registry注册中心](https://cloudland.github.io/assets/images/202106/seata-3.png){:.rounded}

    * 修改`registry`配置, 推荐使用`nacos`。修改`type=nacos`

    * 基于配置`type=nacos`为注册中心, 修改`nacos`配置节点

    ![registry配置中心](https://cloudland.github.io/assets/images/202106/seata-4.png){:.rounded}

    * 修改`config`配置, 推荐使用`nacos`。修改`type=nacos`

    * 基于配置`type=nacos`为配置中心, 修改`nacos`配置节点

### 4. Seata 数据库

#### TC 事物控制者

> 事物控制者, 由`seata-server`负责协调控制。这块内容只有在`file.conf`中配置`type=db`, 方需要做数据库表的初始化工作。

* postgresql

  ```SQL
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

#### RM 事物参与者

> 事物参与者在AT模式下，需要 `undo_log` 建表语句

* postgresql

  ```SQL
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
### 5. Seata 启动

启动命令参数说明:

```
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

```
sh seata-server.sh -p 8091 -h 127.0.0.1 -m db

# 后台运行 重定向日志文件 2>&1 是一个整体，2 表示stderr标准错误，报错内容
nohup sh seata-server.sh -p 8091 -h 127.0.0.1 -m db file.log 2>&1 &
```