> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html)

#### 29.12.13.4 表句柄表

性能模式通过[`table_handles`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html)表公开表锁信息，以显示每个打开的表句柄当前生效的表锁。

[`table_handles`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html)表是只读的，不能更新。默认情况下自动调整大小；要配置表大小，请在服务器启动时设置[`performance_schema_max_table_handles`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-system-variables.html#sysvar_performance_schema_max_table_handles)系统变量。

表锁仪表化使用`wait/lock/table/sql/handler`工具，这是默认启用的。

要在服务器启动时控制表锁仪表化状态，请在`my.cnf`文件中使用以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='wait/lock/table/sql/handler=ON'
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='wait/lock/table/sql/handler=OFF'
    ```

要在运行时控制表锁仪表化状态，请更新[`setup_instruments`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-instruments-table.html)表：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES', TIMED = 'YES'
    WHERE NAME = 'wait/lock/table/sql/handler';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO', TIMED = 'NO'
    WHERE NAME = 'wait/lock/table/sql/handler';
    ```

[`table_handles`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html)表具有以下列：

+   `OBJECT_TYPE`

    由表句柄打开的表。

+   `OBJECT_SCHEMA`

    包含对象的模式。

+   `OBJECT_NAME`

    仪表化对象的名称。

+   `OBJECT_INSTANCE_BEGIN`

    内存中的表句柄地址。

+   `OWNER_THREAD_ID`

    拥有表句柄的线程。

+   `OWNER_EVENT_ID`

    导致打开表句柄的事件。

+   `INTERNAL_LOCK`

    在 SQL 级别使用的表锁。该值是`READ`、`READ WITH SHARED LOCKS`、`READ HIGH PRIORITY`、`READ NO INSERT`、`WRITE ALLOW WRITE`、`WRITE CONCURRENT INSERT`、`WRITE LOW PRIORITY`或`WRITE`中的一个。有关这些锁类型的信息，请参见`include/thr_lock.h`源文件。

+   `EXTERNAL_LOCK`

    在存储引擎级别使用的表锁。该值是`READ EXTERNAL`或`WRITE EXTERNAL`中的一个。

[`table_handles`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html)表具有以下索引：

+   在(`OBJECT_INSTANCE_BEGIN`)上的主键。

+   在(`OBJECT_TYPE`, `OBJECT_SCHEMA`, `OBJECT_NAME`)上的索引。

+   在(`OWNER_THREAD_ID`, `OWNER_EVENT_ID`)上的索引。

[`TRUNCATE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html)语句不允许用于[`table_handles`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-table-handles-table.html)表。
