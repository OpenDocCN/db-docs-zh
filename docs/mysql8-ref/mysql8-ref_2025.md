# 29.12.4 性能模式等待事件表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-wait-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-wait-tables.html)

29.12.4.1 events_waits_current 表

29.12.4.2 events_waits_history 表

29.12.4.3 events_waits_history_long 表

性能模式工具记录等待事件，这些事件需要时间。在事件层次结构中，等待事件嵌套在阶段事件中，阶段事件嵌套在语句事件中，语句事件嵌套在事务事件中。

这些表存储等待事件：

+   `events_waits_current`：每个线程的当前等待事件。

+   `events_waits_history`：每个线程最近结束的等待事件。

+   `events_waits_history_long`：全局最近结束的等待事件（跨所有线程）。

以下各节描述了等待事件表。还有汇总信息关于等待事件的摘要表；请参阅第 29.12.20.1 节，“等待事件摘要表”。

有关三个等待事件表之间关系的更多信息，请参阅第 29.9 节，“当前和历史事件的性能模式表”。

#### 配置等待事件收集

要控制是否收集等待事件，请设置相关工具和消费者的状态：

+   `setup_instruments`表包含以`wait`开头的工具名称。使用这些工具来启用或禁用单个等待事件类的收集。

+   `setup_consumers`表包含与当前和历史等待事件表名称对应的消费者值。使用这些消费者来过滤等待事件的收集。

一些等待工具默认启用；其他则禁用。例如：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'wait/io/file/innodb%';
+-------------------------------------------------+---------+-------+
| NAME                                            | ENABLED | TIMED |
+-------------------------------------------------+---------+-------+
| wait/io/file/innodb/innodb_tablespace_open_file | YES     | YES   |
| wait/io/file/innodb/innodb_data_file            | YES     | YES   |
| wait/io/file/innodb/innodb_log_file             | YES     | YES   |
| wait/io/file/innodb/innodb_temp_file            | YES     | YES   |
| wait/io/file/innodb/innodb_arch_file            | YES     | YES   |
| wait/io/file/innodb/innodb_clone_file           | YES     | YES   |
+-------------------------------------------------+---------+-------+
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'wait/io/socket/%';
+----------------------------------------+---------+-------+
| NAME                                   | ENABLED | TIMED |
+----------------------------------------+---------+-------+
| wait/io/socket/sql/server_tcpip_socket | NO      | NO    |
| wait/io/socket/sql/server_unix_socket  | NO      | NO    |
| wait/io/socket/sql/client_connection   | NO      | NO    |
+----------------------------------------+---------+-------+
```

等待消费者默认情况下处于禁用状态：

```sql
mysql> SELECT *
       FROM performance_schema.setup_consumers
       WHERE NAME LIKE 'events_waits%';
+---------------------------+---------+
| NAME                      | ENABLED |
+---------------------------+---------+
| events_waits_current      | NO      |
| events_waits_history      | NO      |
| events_waits_history_long | NO      |
+---------------------------+---------+
```

要在服务器启动时控制等待事件的收集，请在您的`my.cnf`文件中使用以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='wait/%=ON'
    performance-schema-consumer-events-waits-current=ON
    performance-schema-consumer-events-waits-history=ON
    performance-schema-consumer-events-waits-history-long=ON
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='wait/%=OFF'
    performance-schema-consumer-events-waits-current=OFF
    performance-schema-consumer-events-waits-history=OFF
    performance-schema-consumer-events-waits-history-long=OFF
    ```

要在运行时控制等待事件收集，请更新`setup_instruments`和`setup_consumers`表：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES', TIMED = 'YES'
    WHERE NAME LIKE 'wait/%';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'YES'
    WHERE NAME LIKE 'events_waits%';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO', TIMED = 'NO'
    WHERE NAME LIKE 'wait/%';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'NO'
    WHERE NAME LIKE 'events_waits%';
    ```

要仅收集特定的等待事件，请仅启用相应的等待工具。要仅为特定的等待事件表收集等待事件，请启用等待工具，但仅启用与所需表对应的等待消费者。

有关配置事件收集的更多信息，请参阅第 29.3 节，“性能模式启动配置”和第 29.4 节，“性能模式运行时配置”。
