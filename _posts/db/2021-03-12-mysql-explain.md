---
comment: false
aside:
  toc: true

title: MySQL Explain
date: 2021-03-12 09:00
tags: 数据库 MySQL
---

**Explain 执行结果**

![Explain执行结果](https://cloudland.github.io/assets/images/20210321/explain-01.png)

### id: 查询序列号

* `select`{:.success}查询的序列号, 包含一组数字, 表示查询执行`select`{:.success}操作的表顺序;
* 如果`SQL`存在子查询, **id**值越大优先级越高, 优先被执行;
* 如果`SQL`存在子查询, **id**值相同, 从上往下顺序执行;

### select_type: 查询类型

* `SIMPLE`: 简单查询

  查询中不包含子查询或`UNION`{:.success};

* `PRIMARY`: 主键查询

  查询中包含任何复杂的子查询, 最外层标记;

* `SUBQUERY`: 子查询

  在 `SELECT`{:.success} 或 `WHERE`{:.success} 中包含子查询;

* `DERIVER`: 衍生

  在 `FROM`{:.success} 列表中包含子查询被标记为 **DERIVER**, 结果存放在临时表;
  若 `UNION`{:.success} 包含在 `FOME`{:.success} 子查询中, 外层 `SELECT`{:.success} 将被标记**DERIVER**;

* `UNION`:

  第二个`SELECT`{:.success}出现在`UNION`{:.success}之后;

* `UNION RESULT`:

  从 `UNION`{:.success} 表获取合并结果的 `SELECT`{:.success};

### table: 查询表名

SQL 执行的哪张表

### type: 访问类型

类型优劣顺序 

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

> 一般至少保障到 range, 最好能达到 ref

* `ALL`{:.error}

  全表扫描(Full Table Scan)遍历以找到匹配行

* `index`{:.warning}

  全索引扫描(Full Index Scan)与 ALL 区别为只遍历索引树。

* `range`{:.success}

  用一个索引来检索给定范围的行。

* `ref`{:.success}

  非唯一性索引扫描, 返回匹配某个单独值的所有行。

* `eq_ref`{:.success}

  唯一性索引扫描, 对于索引键表中只有一条记录与之匹配。常出现于主键或唯一索引扫描。

* `const`{:.success}

  通过索引一次就能找到, const 用于 `PrimaryKey`(主键) 或 `Unique`(唯一) 索引。 

* `system`{:.success}

  表只有一行数据(等于系统表), `const`类型特例。

* `NULL`{:.success}

  不用访问表或者索引就直接能到结果

### possible_keys: 可能用到的索引

显示一个或多个可能应用在此次查询的索引, 但不一定被查询实际使用。

### key: 实际使用索引

实际使用的索引, 如果为 NULL 表示没有使用索引。

### key_len: 索引长度 

表示索引中使用的字节数, 可计算出查询中使用的索引长度。在不损失精度的情况下, 长度越短越好。
显示的值为索引字段的最大可能长度, 并非实际长度。是根据表计算而得, 不是通过表检索得出。

### ref: 显示索引使用的列

显示索引使用了表的哪一个列。或者 const 表示使用了一个常量。表示哪些列或者常量被用于查询索引列上的值。

### rows: 预处理行数

根据表统计信息及索引选用情况，预估找到所需记录需要读取的行数。

### Extra: 扩展信息

* `Using filesort`{:.error}: 使用外部索引排序, 而不是表内索引顺序读取

* `Using temporary`{:.error}: 使用临时表保存中间结果, 在对查询结果排序。常见于 `ORDER BY`{:.error} 和 `GROUP BY`{:.error}

* `Using index`{:.success}: 覆盖索引(Covering Index)。表示直接从索引读取数据并未执行查询动作。同事出现 Using where 表示索引被用来执行索引键值的查找。

* `Using where`: 使用 `WHERE`{:.success} 过滤

* `Using join buffer`: 使用连接缓存

* `impossible where`: `WHERE`{:.success} 条件不能用来获取任何内容

* `select tables optimized away`: 在没有 `GROUP BY` 子句的情况下, 基于索引优化 MIN/MAX 操作或对于MyISAM存储引擎优化 COUNT(*) 操作, 不必等到执行阶段, 查询执行计划生成阶段即完成优化

* `distinct`: 在找到第一个匹配元素后即停止找同样值的动作



<!--more-->