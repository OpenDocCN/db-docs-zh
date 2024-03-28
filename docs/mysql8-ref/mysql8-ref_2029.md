# 29.12.5 性能模式阶段事件表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-stage-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-stage-tables.html)

29.12.5.1 events_stages_current 表

29.12.5.2 events_stages_history 表

29.12.5.3 events_stages_history_long 表

Performance Schema 仪器阶段，这些是语句执行过程中的步骤，例如解析语句、打开表或执行`filesort`操作。阶段对应于由 `SHOW PROCESSLIST` 显示的线程状态，或者在信息模式 `PROCESSLIST` 表中可见的状态。阶段在状态值更改时开始和结束。

在事件层次结构中，等待事件嵌套在阶段事件中，阶段事件嵌套在语句事件中，语句事件嵌套在事务事件中。

这些表存储阶段事件：

+   `events_stages_current`：每个线程的当前阶段事件。

+   `events_stages_history`：每个线程已结束的最近阶段事件。

+   `events_stages_history_long`：全局已结束的最近阶段事件（跨所有线程）。

以下各节描述了阶段事件表。还有汇总信息的摘要表，汇总了有关阶段事件的信息；请参见 第 29.12.20.2 节，“阶段摘要表”。

有关三个阶段事件表之间关系的更多信息，请参见 第 29.9 节，“当前和历史事件的性能模式表”。

+   配置阶段事件收集

+   阶段事件进度信息

#### 配置阶段事件收集

要控制是否收集阶段事件，请设置相关仪器和消费者的状态：

+   `setup_instruments` 包含以`stage`开头的仪器。使用这些仪器来启用或禁用单个阶段事件类的收集。

+   `setup_consumers`表包含与当前和历史阶段事件表名称对应的消费者值。使用这些消费者来过滤阶段事件的收集。

除了提供语句进度信息的仪器外，阶段仪器默认情况下是禁用的。例如：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE NAME RLIKE 'stage/sql/[a-c]';
+----------------------------------------------------+---------+-------+
| NAME                                               | ENABLED | TIMED |
+----------------------------------------------------+---------+-------+
| stage/sql/After create                             | NO      | NO    |
| stage/sql/allocating local table                   | NO      | NO    |
| stage/sql/altering table                           | NO      | NO    |
| stage/sql/committing alter table to storage engine | NO      | NO    |
| stage/sql/Changing master                          | NO      | NO    |
| stage/sql/Checking master version                  | NO      | NO    |
| stage/sql/checking permissions                     | NO      | NO    |
| stage/sql/cleaning up                              | NO      | NO    |
| stage/sql/closing tables                           | NO      | NO    |
| stage/sql/Connecting to master                     | NO      | NO    |
| stage/sql/converting HEAP to MyISAM                | NO      | NO    |
| stage/sql/Copying to group table                   | NO      | NO    |
| stage/sql/Copying to tmp table                     | NO      | NO    |
| stage/sql/copy to tmp table                        | NO      | NO    |
| stage/sql/Creating sort index                      | NO      | NO    |
| stage/sql/creating table                           | NO      | NO    |
| stage/sql/Creating tmp table                       | NO      | NO    |
+----------------------------------------------------+---------+-------+
```

默认情况下启用并计时提供语句进度信息的阶段事件仪器：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE ENABLED='YES' AND NAME LIKE "stage/%";
+------------------------------------------------------+---------+-------+
| NAME                                                 | ENABLED | TIMED |
+------------------------------------------------------+---------+-------+
| stage/sql/copy to tmp table                          | YES     | YES   |
| stage/sql/Applying batch of row changes (write)      | YES     | YES   |
| stage/sql/Applying batch of row changes (update)     | YES     | YES   |
| stage/sql/Applying batch of row changes (delete)     | YES     | YES   |
| stage/innodb/alter table (end)                       | YES     | YES   |
| stage/innodb/alter table (flush)                     | YES     | YES   |
| stage/innodb/alter table (insert)                    | YES     | YES   |
| stage/innodb/alter table (log apply index)           | YES     | YES   |
| stage/innodb/alter table (log apply table)           | YES     | YES   |
| stage/innodb/alter table (merge sort)                | YES     | YES   |
| stage/innodb/alter table (read PK and internal sort) | YES     | YES   |
| stage/innodb/buffer pool load                        | YES     | YES   |
| stage/innodb/clone (file copy)                       | YES     | YES   |
| stage/innodb/clone (redo copy)                       | YES     | YES   |
| stage/innodb/clone (page copy)                       | YES     | YES   |
+------------------------------------------------------+---------+-------+
```

阶段消费者默认情况下是禁用的：

```sql
mysql> SELECT *
       FROM performance_schema.setup_consumers
       WHERE NAME LIKE 'events_stages%';
+----------------------------+---------+
| NAME                       | ENABLED |
+----------------------------+---------+
| events_stages_current      | NO      |
| events_stages_history      | NO      |
| events_stages_history_long | NO      |
+----------------------------+---------+
```

要在服务器启动时控制阶段事件收集，请在您的`my.cnf`文件中使用以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='stage/%=ON'
    performance-schema-consumer-events-stages-current=ON
    performance-schema-consumer-events-stages-history=ON
    performance-schema-consumer-events-stages-history-long=ON
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='stage/%=OFF'
    performance-schema-consumer-events-stages-current=OFF
    performance-schema-consumer-events-stages-history=OFF
    performance-schema-consumer-events-stages-history-long=OFF
    ```

要在运行时控制阶段事件收集，请更新`setup_instruments`和`setup_consumers`表：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES', TIMED = 'YES'
    WHERE NAME LIKE 'stage/%';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'YES'
    WHERE NAME LIKE 'events_stages%';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO', TIMED = 'NO'
    WHERE NAME LIKE 'stage/%';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'NO'
    WHERE NAME LIKE 'events_stages%';
    ```

仅收集特定阶段事件，只需启用相应的阶段仪器。要仅为特定阶段事件表收集阶段事件，请启用阶段仪器，但仅启用与所需表对应的阶段消费者。

有关配置事件收集的其他信息，请参见第 29.3 节，“Performance Schema 启动配置”和第 29.4 节，“Performance Schema 运行时配置”。

#### 阶段事件进度信息

Performance Schema 阶段事件表包含两列，这两列一起为每行提供阶段进度指示器：

+   `WORK_COMPLETED`：阶段完成的工作单元数量

+   `WORK_ESTIMATED`：阶段预期的工作单元数量

如果没有为仪器提供进度信息，则每列均为`NULL`。如果可用，信息的解释完全取决于仪器实现。Performance Schema 表提供了存储进度数据的容器，但不对度量本身的语义做任何假设：

+   “工作单元”是一个整数度量，随着执行时间的增加而增加，例如处理的字节数、行数、文件数或表数。对于特定仪器的“工作单元”的定义由提供数据的仪器代码确定。

+   `WORK_COMPLETED`值可以根据被仪器化的代码一次或多次增加一个单位。

+   `WORK_ESTIMATED`值可以在阶段期间更改，具体取决于被仪器化的代码。

用于阶段事件进度指示器的仪器可以实现以下任何行为：

+   无进度仪器

    这是最典型的情况，即没有提供进度数据。`WORK_COMPLETED`和`WORK_ESTIMATED`列都为`NULL`。

+   无限进度仪器

    只有`WORK_COMPLETED`列是有意义的。`WORK_ESTIMATED`列没有提供任何数据，显示为 0。

    通过查询监视会话的`events_stages_current`表，监视应用程序可以报告到目前为止完成了多少工作，但无法报告阶段是否接近完成。目前没有类似这样的阶段被仪器化。

+   有界进度仪器

    `WORK_COMPLETED`和`WORK_ESTIMATED`列都是有意义的。

    这种类型的进度指示器适用于具有定义完成标准的操作，例如后面描述的表复制工具。通过查询监视会话的`events_stages_current`表，监视应用程序可以报告到目前为止完成了多少工作，并且可以通过计算`WORK_COMPLETED` / `WORK_ESTIMATED`比率来报告阶段的整体完成百分比。

`stage/sql/copy to tmp table`工具展示了进度指示器的工作原理。在执行`ALTER TABLE`语句期间，会使用`stage/sql/copy to tmp table`阶段，这个阶段的执行时间可能会很长，取决于要复制的数据大小。

表复制任务有一个定义的终止条件（所有行都已复制），`stage/sql/copy to tmp table`阶段被设计为提供有界的进度信息：使用的工作单元是复制的行数，`WORK_COMPLETED`和`WORK_ESTIMATED`都是有意义的，它们的比率表示任务完成的百分比。

要启用该仪器和相关的消费者，请执行以下语句：

```sql
UPDATE performance_schema.setup_instruments
SET ENABLED='YES'
WHERE NAME='stage/sql/copy to tmp table';

UPDATE performance_schema.setup_consumers
SET ENABLED='YES'
WHERE NAME LIKE 'events_stages_%';
```

要查看正在进行的`ALTER TABLE`语句的进度，请从`events_stages_current`表中进行选择。
