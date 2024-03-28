# 29.12.7 性能模式事务表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html)

[29.12.7.1 事件 _ 事务 _ 当前表](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html)

[29.12.7.2 事件 _ 事务 _ 历史表](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-table.html)

[29.12.7.3 事件 _ 事务 _ 历史 _ 长表](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-long-table.html)

性能模式对事务进行仪表化。在事件层次结构中，等待事件嵌套在阶段事件中，阶段事件嵌套在语句事件中，语句事件嵌套在事务事件中。

这些表存储事务事件：

+   [`events_transactions_current`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-current-table.html "29.12.7.1 事件 _ 事务 _ 当前表"): 每个线程的当前事务事件。

+   [`events_transactions_history`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-table.html "29.12.7.2 事件 _ 事务 _ 历史表"): 每个线程结束的最近事务事件。

+   [`events_transactions_history_long`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-events-transactions-history-long-table.html "29.12.7.3 事件 _ 事务 _ 历史 _ 长表"): 全局结束的最近事务事件（跨所有线程）。

以下部分描述了事务事件表。还有汇总表汇总有关事务事件的信息；参见[第 29.12.20.5 节，“事务汇总表”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-summary-tables.html "29.12.20.5 事务汇总表")。

有关三个事务事件表之间关系的更多信息，请参见[第 29.9 节，“当前和历史事件的性能模式表”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-event-tables.html "29.9 当��和历史事件的性能模式表")。

+   [配置事务事件收集](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html#performance-schema-transaction-tables-configuration "Configuring Transaction Event Collection")

+   [事务边界](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html#performance-schema-transaction-tables-transaction-boundaries "Transaction Boundaries")

+   [事务仪表化](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html#performance-schema-transaction-tables-instrumentation "Transaction Instrumentation")

+   [事务和嵌套事件](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html#performance-schema-transaction-tables-nested-events "Transactions and Nested Events")

+   [事务和存储过程](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html#performance-schema-transaction-tables-stored-programs "Transactions and Stored Programs")

+   [事务和保存点](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-transaction-tables.html#performance-schema-transaction-tables-savepoints "Transactions and Savepoints")

+   事务和错误

#### 配置事务事件收集

要控制是否收集事务事件，请设置相关工具和消费者的状态：

+   `setup_instruments`表包含一个名为`transaction`的工具。使用此工具来启用或禁用单个事务事件类的收集。

+   `setup_consumers`表包含与当前和历史事务事件表名称对应的消费者值。使用这些消费者来过滤事务事件的收集。

`transaction`工具和`events_transactions_current`以及`events_transactions_history`事务消费者默认启用：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE NAME = 'transaction';
+-------------+---------+-------+
| NAME        | ENABLED | TIMED |
+-------------+---------+-------+
| transaction | YES     | YES   |
+-------------+---------+-------+
mysql> SELECT *
       FROM performance_schema.setup_consumers
       WHERE NAME LIKE 'events_transactions%';
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | NO      |
+----------------------------------+---------+
```

要在服务器启动时控制事务事件的收集，请在您的`my.cnf`文件中使用类似以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='transaction=ON'
    performance-schema-consumer-events-transactions-current=ON
    performance-schema-consumer-events-transactions-history=ON
    performance-schema-consumer-events-transactions-history-long=ON
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='transaction=OFF'
    performance-schema-consumer-events-transactions-current=OFF
    performance-schema-consumer-events-transactions-history=OFF
    performance-schema-consumer-events-transactions-history-long=OFF
    ```

要在运行时控制事务事件的收集，请更新`setup_instruments`和`setup_consumers`表：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES', TIMED = 'YES'
    WHERE NAME = 'transaction';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'YES'
    WHERE NAME LIKE 'events_transactions%';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO', TIMED = 'NO'
    WHERE NAME = 'transaction';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'NO'
    WHERE NAME LIKE 'events_transactions%';
    ```

为了仅收集特定事务事件表的事务事件，启用`transaction`工具，但仅启用与所需表对应的事务消费者。

有关配置事件收集的其他信息，请参见第 29.3 节“性能模式启动配置”和第 29.4 节“性能模式运行时配置”。

#### 事务边界

在 MySQL 服务器中，事务明确以以下语句开始：

```sql
START TRANSACTION | BEGIN | XA START | XA BEGIN
```

事务也会隐式开始。例如，当启用`autocommit`系统变量时，每个语句的开始都会启动一个新事务。

当禁用`autocommit`时，提交事务后的第一条语句标志着新事务的开始。直到提交为止，后续语句都属于该事务。

事务明确以以下语句结束：

```sql
COMMIT | ROLLBACK | XA COMMIT | XA ROLLBACK
```

事务也会隐式结束，通过 DDL 语句、锁定语句和服务器管理语句的执行。

在下面的讨论中，对`START TRANSACTION`的引用也适用于`BEGIN`，`XA START`和`XA BEGIN`。同样，对`COMMIT`和`ROLLBACK`的引用也适用于`XA COMMIT`和`XA ROLLBACK`。

性能模式定义事务边界与服务器类似。事务事件的开始和结束与服务器中相应的状态转换非常相似：

+   对于显式启动的事务，事务事件在处理`START TRANSACTION`语句期间开始。

+   对于隐式启动的事务，事务事件从上一个事务结束后第一次使用事务引擎的语句开始。

+   对于任何事务，无论是显式还是隐式结束，事务事件在服务器在处理`COMMIT`或`ROLLBACK`期间转换出活动事务状态时结束。

这种方法有微妙的含义：

+   性能模式中的事务事件并不完全包括与相应的`START TRANSACTION`，`COMMIT`或`ROLLBACK`语句相关联的语句事件。事务事件与这些语句之间存在微小的时间重叠。

+   使用非事务引擎的语句对连接的事务状态没有影响。对于隐式事务，事务事件从第一个使用事务引擎的语句开始。这意味着仅在非事务表上操作的语句将被忽略，即使在`START TRANSACTION`之后也是如此。

举例说明，考虑以下情景：

```sql
1\. SET autocommit = OFF;
2\. CREATE TABLE t1 (a INT) ENGINE = InnoDB;
3\. START TRANSACTION;                       -- Transaction 1 START
4\. INSERT INTO t1 VALUES (1), (2), (3);
5\. CREATE TABLE t2 (a INT) ENGINE = MyISAM; -- Transaction 1 COMMIT
                                            -- (implicit; DDL forces commit)
6\. INSERT INTO t2 VALUES (1), (2), (3);     -- Update nontransactional table
7\. UPDATE t2 SET a = a + 1;                 -- ... and again
8\. INSERT INTO t1 VALUES (4), (5), (6);     -- Write to transactional table
                                            -- Transaction 2 START (implicit)
9\. COMMIT;                                  -- Transaction 2 COMMIT
```

从服务器的角度看，事务 1 在表`t2`创建时结束。事务 2 直到访问事务表时才开始，尽管在此期间更新了非事务表。

从性能模式的角度看，当服务器转换为活动事务状态时，事务 2 开始。语句 6 和 7 不包括在事务 2 的范围内，这与服务器将事务写入二进制日志的方式一致。

#### 事务仪表化

三个属性定义了事务：

+   访问模式（只读，读写）

+   隔离级别（`SERIALIZABLE`，`REPEATABLE READ`，等等）

+   隐式（启用`autocommit`）或显式（禁用`autocommit`）

为了减少事务仪表化的复杂性，并确保收集的事务数据提供完整、有意义的结果，所有事务都是独立于访问模式、隔离级别或自动提交模式进行仪表化的。

要选择性地检查事务历史记录，请使用事务事件表中的属性列：`ACCESS_MODE`，`ISOLATION_LEVEL`和`AUTOCOMMIT`。

可以通过各种方式减少事务仪表化的成本，例如根据用户、账户、主机或线程（客户端连接）启用或禁用事务仪表化。

#### 事务和嵌套事件

事务事件的父级是启动事务的事件。对于显式启动的事务，这包括`START TRANSACTION`和`COMMIT AND CHAIN`语句。对于隐式启动的事务，它是在上一个事务结束后第一个使用事务引擎的语句。

一般来说，事务是在事务期间启动的所有事件的最高级父级，包括明确结束事务的语句，比如`COMMIT`和`ROLLBACK`。异常情况是隐式结束事务的语句，比如 DDL 语句，在这种情况下，必须在执行新语句之前提交当前事务。

#### 事务和存储程序

事务和存储程序事件相关如下：

+   存储过程

    存储过程独立于事务运行。存储过程可以在事务内启动，并且可以在存储过程内启动或结束事务。如果在事务内调用，存储过程可以执行强制提交父事务的语句，然后启动新事务。

    如果在事务内启动存储过程，则该事务是存储过程事件的父级。

    如果事务是由存储过程启动的，则存储过程是事务事件的父级。

+   存储函数

    存储函数受限于不引起显式或隐式提交或回滚。存储函数事件可以存在于父事务事件中。

+   触发器

    触发器作为访问与其关联的表的语句的一部分而激活，因此触发器事件的父级始终是激活它的语句。

    触发器不能发出导致事务显式或隐式提交或回滚的语句。

+   计划事件

    计划事件体中语句的执行发生在一个新连接中。计划事件嵌套在父事务中不适用。

#### 事务和保存点

保存点语句被记录为单独的语句事件。事务事件包括在事务期间发出的`SAVEPOINT`、`ROLLBACK TO SAVEPOINT`和`RELEASE SAVEPOINT`语句的单独计数器。

#### 事务和错误

在事务中发生的错误和警告记录在语句事件中，而不记录在相应的事务事件中。这包括事务特定的错误和警告，例如在非事务表上的回滚或 GTID 一致性错误。
