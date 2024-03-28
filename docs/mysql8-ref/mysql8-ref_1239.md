# 17.12.2 在线 DDL 性能和并发性

> 译文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-performance.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-performance.html)

在线 DDL 改进了 MySQL 操作的几个方面：

+   访问表的应用程序更具响应性，因为在 DDL 操作进行时，表上的查询和 DML 操作可以继续进行。减少对 MySQL 服务器资源的锁定和等待会导致更大的可伸缩性，即使对于不涉及 DDL 操作的操作也是如此。

+   即时操作仅修改数据字典中的元数据。在操作执行阶段可能会短暂地获取表的独占元数据锁。表数据不受影响，使操作瞬间完成。允许并发 DML。

+   在线操作避免了与表复制方法相关的磁盘 I/O 和 CPU 周期，从而最小化了数据库的整体负载。减少负载有助于在 DDL 操作期间保持良好的性能和高吞吐量。

+   在线操作读取的数据比表复制操作少，这减少了频繁访问数据从内存中清除的次数。频繁访问数据的清除可能导致 DDL 操作后的临时性能下降。

#### LOCK 子句

默认情况下，MySQL 在 DDL 操作期间尽可能少地使用锁定。如果需要，可以为原地操作和一些复制操作指定`LOCK`子句以强制执行更严格的锁定。如果`LOCK`子句指定的锁定级别比特定 DDL 操作允许的锁定级别更不严格，则语句将失败并显示错误。下面按照从最不严格到最严格的顺序描述了`LOCK`子句： 

+   `LOCK=NONE`:

    允许并发查询和 DML。

    例如，对涉及客户注册或购买的表使用此子句，以避免在长时间的 DDL 操作期间使表不可用。

+   `LOCK=SHARED`:

    允许并发查询但阻止 DML。

    例如，在数据仓库表上使用此子句，可以延迟数据加载操作直到 DDL 操作完成，但查询不能被延迟太长时间。

+   `LOCK=DEFAULT`:

    允许尽可能多的并发性（并发查询、DML 或两者兼有）。省略`LOCK`子句与指定`LOCK=DEFAULT`相同。

    当您不希望 DDL 语句的默认锁定级别对表造成任何可用性问题时，请使用此子句。

+   `LOCK=EXCLUSIVE`:

    阻止并发查询和 DML。

    如果主要关注尽快完成 DDL 操作，并且不需要并发查询和 DML 访问，则使用此子句。如果服务器应该处于空闲状态，以避免意外的表访问，也可以使用此子句。

#### 在线 DDL 和元数据锁

在线 DDL 操作可以看作有三个阶段：

+   *阶段 1：初始化*

    在初始化阶段，服务器确定操作期间允许多少并发性，考虑到存储引擎的能力、语句中指定的操作以及用户指定的`ALGORITHM`和`LOCK`选项。在此阶段，会获取一个共享可升级的元数据锁以保护当前表定义。

+   *阶段 2: 执行*

    在此阶段，语句被准备和执行。元数据锁是否升级为独占取决于初始化阶段评估的因素。如果需要独占元数据锁，则仅在语句准备期间短暂获取。

+   *阶段 3: 提交表定义*

    在提交表定义阶段，元数据锁被升级为独占，以清除旧表定义并提交新表定义。一旦授予，独占元数据锁的持续时间很短。

由于上述独占元数据锁要求，在线 DDL 操作可能需要等待对表上持有元数据锁的并发事务进行提交或回滚。在 DDL 操作之前或期间启动的事务可以在正在更改的表上持有元数据锁。在长时间运行或不活动事务的情况下，在线 DDL 操作可能会因等待独占元数据锁而超时。此外，在线 DDL 操作请求的待处理独占元数据锁会阻止表上的后续事务。

以下示例演示了一个在线 DDL 操作等待独占元数据锁的情况，以及待处理元数据锁如何阻止表上的后续事务。

会话 1:

```sql
mysql> CREATE TABLE t1 (c1 INT) ENGINE=InnoDB;
mysql> START TRANSACTION;
mysql> SELECT * FROM t1;
```

会话 1 的`SELECT`语句在表 t1 上获取了一个共享元数据锁。

会话 2:

```sql
mysql> ALTER TABLE t1 ADD COLUMN x INT, ALGORITHM=INPLACE, LOCK=NONE;
```

会话 2 中的在线 DDL 操作需要在表 t1 上获取独占元数据锁以提交表定义更改，必须等待会话 1 的事务提交或回滚。

会话 3:

```sql
mysql> SELECT * FROM t1;
```

会话 3 中发出的`SELECT`语句正在等待会话 2 中的`ALTER TABLE`操作请求的独占元数据锁被授予。

您可以使用`SHOW FULL PROCESSLIST`来确定事务是否在等待元数据锁。

```sql
mysql> SHOW FULL PROCESSLIST\G
...
*************************** 2\. row ***************************
     Id: 5
   User: root
   Host: localhost
     db: test
Command: Query
   Time: 44
  State: Waiting for table metadata lock
   Info: ALTER TABLE t1 ADD COLUMN x INT, ALGORITHM=INPLACE, LOCK=NONE
...
*************************** 4\. row ***************************
     Id: 7
   User: root
   Host: localhost
     db: test
Command: Query
   Time: 5
  State: Waiting for table metadata lock
   Info: SELECT * FROM t1 4 rows in set (0.00 sec)
```

元数据锁信息也通过性能模式`metadata_locks`表暴露出来，该表提供了关于会话之间的元数据锁依赖关系、会话正在等待的元数据锁以及当前持有元数据锁的会话的信息。更多信息，请参见 Section 29.12.13.3, “The metadata_locks Table”。

#### 在线 DDL 性能

DDL 操作的性能在很大程度上取决于操作是立即执行、原地执行还是重建表。

要评估 DDL 操作的相对性能，可以比较使用`ALGORITHM=INSTANT`、`ALGORITHM=INPLACE`和`ALGORITHM=COPY`的结果。还可以启用`old_alter_table`运行语句以强制使用`ALGORITHM=COPY`。

对于修改表数据的 DDL 操作，您可以通过查看命令完成后显示的“受影响行数”值来确定 DDL 操作是在原地执行还是执行表复制。例如：

+   更改列的默认值（快速，不影响表数据）：

    ```sql
    Query OK, 0 rows affected (0.07 sec)
    ```

+   添加索引（需要时间，但`0 rows affected`表明表没有被复制）：

    ```sql
    Query OK, 0 rows affected (21.42 sec)
    ```

+   更改列的数据类型（需要大量时间，并需要重建表的所有行）：

    ```sql
    Query OK, 1671168 rows affected (1 min 35.54 sec)
    ```

在大表上运行 DDL 操作之前，请按以下方式检查操作是快还是慢：

1.  克隆表结构。

1.  使用少量数据填充克隆表。

1.  在克隆表上运行 DDL 操作。

1.  检查“受影响行数”值是否为零。非零值表示操作复制表数据，这可能需要特殊规划。例如，您可以在计划的停机期间执行 DDL 操作，或者逐个在每个副本服务器上执行。

注意

要更好地了解与 DDL 操作相关的 MySQL 处理，可以在 DDL 操作之前和之后检查与`InnoDB`相关的性能模式和`INFORMATION_SCHEMA`表，以查看物理读取、写入、内存分配等的数量。

可以使用性能模式阶段事件来监视`ALTER TABLE`的进度。请参阅第 17.16.1 节，“使用性能模式监视 InnoDB 表的 ALTER TABLE 进度”。

由于记录并发 DML 操作所做的更改涉及一些处理工作，然后在最后应用这些更改，因此在线 DDL 操作的总体时间可能比阻止其他会话访问表的表复制机制更长。原始性能的降低与对使用表的应用程序更好的响应性之间取得平衡。在评估更改表结构的技术时，考虑基于诸如网页加载时间等因素的终端用户对性能的感知。
