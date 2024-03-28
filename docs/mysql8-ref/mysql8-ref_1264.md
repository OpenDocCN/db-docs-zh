# 17.16.1 监视使用性能模式的 InnoDB 表的`ALTER TABLE` 进度

> 原文：[`dev.mysql.com/doc/refman/8.0/en/monitor-alter-table-performance-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/monitor-alter-table-performance-schema.html)

您可以使用性能模式监视`InnoDB`表的`ALTER TABLE`进度。

有七个阶段事件代表`ALTER TABLE` 的不同阶段。每个阶段事件报告了整体`ALTER TABLE` 操作的`WORK_COMPLETED`和`WORK_ESTIMATED`的累计值，随着其在不同阶段的进行而不断更新。`WORK_ESTIMATED`是使用一个公式计算的，该公式考虑了`ALTER TABLE` 执行的所有工作，并且可能在`ALTER TABLE` 处理过程中进行修订。`WORK_COMPLETED`和`WORK_ESTIMATED`的值是对`ALTER TABLE` 执行的所有工作的抽象表示。

按顺序，`ALTER TABLE` 阶段事件包括：

+   `stage/innodb/alter table (read PK and internal sort)`: 当`ALTER TABLE`处于读取主键阶段时，此阶段处于活动状态。它从`WORK_COMPLETED=0`开始，并将`WORK_ESTIMATED`设置为主键中估计的页面数。当阶段完成时，将更新`WORK_ESTIMATED`为主键中实际的页面数。

+   `stage/innodb/alter table (merge sort)`: 此阶段针对`ALTER TABLE`操作添加的每个索引重复执行。

+   `stage/innodb/alter table (insert)`: 此阶段针对`ALTER TABLE`操作添加的每个索引重复执行。

+   `stage/innodb/alter table (log apply index)`: 此阶段包括应用在运行`ALTER TABLE`时生成的 DML 日志。

+   `stage/innodb/alter table (flush)`: 在此阶段开始之前，根据刷新列表的长度，更新`WORK_ESTIMATED`以获得更准确的估计。

+   `stage/innodb/alter table (log apply table)`: 此阶段包括应用在运行`ALTER TABLE`时生成的并发 DML 日志。此阶段的持续时间取决于表更改的程度。如果表上没有运行并发 DML，则此阶段是瞬时的。

+   `stage/innodb/alter table (end)`: 包括在刷新阶段之后出现的任何剩余工作，例如在`ALTER TABLE`运行时对表执行的 DML 的重新应用。

注意

`InnoDB`的`ALTER TABLE`阶段事件目前不考虑空间索引的添加。

#### 使用性能模式监控`ALTER TABLE`的示例

以下示例演示了如何启用`stage/innodb/alter table%`阶段事件工具和相关消费者表来监控`ALTER TABLE`的进度。有关性能模式阶段事件工具和相关消费者的信息，请参阅第 29.12.5 节，“性能模式阶段事件表”。

1.  启用`stage/innodb/alter%`工具：

    ```sql
    mysql> UPDATE performance_schema.setup_instruments
           SET ENABLED = 'YES'
           WHERE NAME LIKE 'stage/innodb/alter%';
    Query OK, 7 rows affected (0.00 sec)
    Rows matched: 7  Changed: 7  Warnings: 0
    ```

1.  启用包括`events_stages_current`、`events_stages_history`和`events_stages_history_long`的阶段事件消费者表。

    ```sql
    mysql> UPDATE performance_schema.setup_consumers
           SET ENABLED = 'YES'
           WHERE NAME LIKE '%stages%';
    Query OK, 3 rows affected (0.00 sec)
    Rows matched: 3  Changed: 3  Warnings: 0
    ```

1.  执行一个`ALTER TABLE`操作。在此示例中，向 employees 示例数据库的 employees 表中添加一个`middle_name`列。

    ```sql
    mysql> ALTER TABLE employees.employees ADD COLUMN middle_name varchar(14) AFTER first_name;
    Query OK, 0 rows affected (9.27 sec)
    Records: 0  Duplicates: 0  Warnings: 0
    ```

1.  通过查询性能模式`events_stages_current`表来检查`ALTER TABLE`操作的进度。显示的阶段事件取决于当前正在进行的`ALTER TABLE`阶段。`WORK_COMPLETED`列显示已完成的工作。`WORK_ESTIMATED`列提供了剩余工作的估计。

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED
           FROM performance_schema.events_stages_current;
    +------------------------------------------------------+----------------+----------------+
    | EVENT_NAME                                           | WORK_COMPLETED | WORK_ESTIMATED |
    +------------------------------------------------------+----------------+----------------+
    | stage/innodb/alter table (read PK and internal sort) |            280 |           1245 |
    +------------------------------------------------------+----------------+----------------+
    1 row in set (0.01 sec)
    ```

    如果`ALTER TABLE`操作已完成，则`events_stages_current`表将返回一个空集。在这种情况下，您可以查询`events_stages_history`表来查看已完成操作的事件数据。例如：

    ```sql
    mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED
           FROM performance_schema.events_stages_history;
    +------------------------------------------------------+----------------+----------------+
    | EVENT_NAME                                           | WORK_COMPLETED | WORK_ESTIMATED |
    +------------------------------------------------------+----------------+----------------+
    | stage/innodb/alter table (read PK and internal sort) |            886 |           1213 |
    | stage/innodb/alter table (flush)                     |           1213 |           1213 |
    | stage/innodb/alter table (log apply table)           |           1597 |           1597 |
    | stage/innodb/alter table (end)                       |           1597 |           1597 |
    | stage/innodb/alter table (log apply table)           |           1981 |           1981 |
    +------------------------------------------------------+----------------+----------------+
    5 rows in set (0.00 sec)
    ```

    如上所示，在`ALTER TABLE`处理过程中`WORK_ESTIMATED`值已经修订。初始阶段完成后的估计工作量为 1213。当`ALTER TABLE`处理完成时，`WORK_ESTIMATED`被设置为实际值，即 1981。
