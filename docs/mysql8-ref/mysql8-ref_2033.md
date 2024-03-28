# 29.12.6 性能模式语句事件表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-tables.html)

29.12.6.1 events_statements_current 表

29.12.6.2 events_statements_history 表

29.12.6.3 events_statements_history_long 表

29.12.6.4 prepared_statements_instances 表

性能模式仪器语句执行。语句事件发生在事件层次结构的高级别。在事件层次结构中，等待事件嵌套在阶段事件中，阶段事件嵌套在语句事件中，语句事件嵌套在事务事件中。

这些表存储语句事件：

+   `events_statements_current`: 每个线程的当前语句事件。

+   `events_statements_history`: 最近每个线程结束的语句事件。

+   `events_statements_history_long`: 最近全局结束的语句事件（跨所有线程）。

+   `prepared_statements_instances`: 准备语句实例和统计信息

以下各节描述了语句事件表。还有汇总信息关于语句事件的表；请参阅 第 29.12.20.3 节，“语句汇总表”。

有关三个 `events_statements_*`xxx`*` 事件表之间关系的更多信息，请参阅 第 29.9 节，“当前和历史事件的性能模式表”。

+   配置语句事件收集

+   语句监控

#### 配置语句事件收集

要控制是否收集语句事件，请设置相关仪器和消费者的状态：

+   `setup_instruments`表包含以`statement`开头的工具名称。使用这些工具来启用或禁用单个语句事件类的收集。

+   `setup_consumers`表包含与当前和历史语句事件表名称以及语句摘要消费者对应的消费者值。使用这些消费者来过滤语句事件和语句摘要的收集。

语句工具默认启用，并且默认启用`events_statements_current`、`events_statements_history`和`statements_digest`语句消费者：

```sql
mysql> SELECT NAME, ENABLED, TIMED
       FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'statement/%';
+---------------------------------------------+---------+-------+
| NAME                                        | ENABLED | TIMED |
+---------------------------------------------+---------+-------+
| statement/sql/select                        | YES     | YES   |
| statement/sql/create_table                  | YES     | YES   |
| statement/sql/create_index                  | YES     | YES   |
...
| statement/sp/stmt                           | YES     | YES   |
| statement/sp/set                            | YES     | YES   |
| statement/sp/set_trigger_field              | YES     | YES   |
| statement/scheduler/event                   | YES     | YES   |
| statement/com/Sleep                         | YES     | YES   |
| statement/com/Quit                          | YES     | YES   |
| statement/com/Init DB                       | YES     | YES   |
...
| statement/abstract/Query                    | YES     | YES   |
| statement/abstract/new_packet               | YES     | YES   |
| statement/abstract/relay_log                | YES     | YES   |
+---------------------------------------------+---------+-------+
```

```sql
mysql> SELECT *
       FROM performance_schema.setup_consumers
       WHERE NAME LIKE '%statements%';
+--------------------------------+---------+
| NAME                           | ENABLED |
+--------------------------------+---------+
| events_statements_current      | YES     |
| events_statements_history      | YES     |
| events_statements_history_long | NO      |
| statements_digest              | YES     |
+--------------------------------+---------+
```

要在服务器启动时控制语句事件收集，请在您的`my.cnf`文件中使用以下行：

+   启用：

    ```sql
    [mysqld]
    performance-schema-instrument='statement/%=ON'
    performance-schema-consumer-events-statements-current=ON
    performance-schema-consumer-events-statements-history=ON
    performance-schema-consumer-events-statements-history-long=ON
    performance-schema-consumer-statements-digest=ON
    ```

+   禁用：

    ```sql
    [mysqld]
    performance-schema-instrument='statement/%=OFF'
    performance-schema-consumer-events-statements-current=OFF
    performance-schema-consumer-events-statements-history=OFF
    performance-schema-consumer-events-statements-history-long=OFF
    performance-schema-consumer-statements-digest=OFF
    ```

要在运行时控制语句事件收集，请更新`setup_instruments`和`setup_consumers`表：

+   启用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'YES', TIMED = 'YES'
    WHERE NAME LIKE 'statement/%';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'YES'
    WHERE NAME LIKE '%statements%';
    ```

+   禁用：

    ```sql
    UPDATE performance_schema.setup_instruments
    SET ENABLED = 'NO', TIMED = 'NO'
    WHERE NAME LIKE 'statement/%';

    UPDATE performance_schema.setup_consumers
    SET ENABLED = 'NO'
    WHERE NAME LIKE '%statements%';
    ```

仅启用相应的语句工具以收集特定语句事件。要仅为特定语句事件表收集语句事件，请启用语句工具，但仅启用与所需表对应的语句消费者。

有关配置事件收集的其他信息，请参见第 29.3 节，“性能模式启动配置”和第 29.4 节，“性能模式运行时配置”。

#### 语句监控

语句监控从服务器看到线程请求活动的时刻开始，直到所有活动停止的时刻结束。通常，这意味着从服务器收到客户端的第一个数据包到服务器完成发送响应的时间。存储程序中的语句与其他语句一样进行监视。

当性能模式工具请求（服务器命令或 SQL 语句）时，它使用从更一般（或“抽象”）到更具体的阶段逐渐进行的工具名称，直到到达最终工具名称。

最终工具名称对应于服务器命令和 SQL 语句：

+   服务器命令对应于`mysql_com.h`头文件中定义的`COM_*`xxx`* codes`，并在`sql/sql_parse.cc`中进行处理。例如`COM_PING`和`COM_QUIT`。命令的工具具有以`statement/com`开头的名称，例如`statement/com/Ping`和`statement/com/Quit`。

+   SQL 语句以文本形式表示，例如`DELETE FROM t1`或`SELECT * FROM t2`。SQL 语句的仪器名称以`statement/sql`开头，例如`statement/sql/delete`和`statement/sql/select`。

一些最终的仪器名称是特定于错误处理的：

+   `statement/com/Error`用于处理服务器接收到的超出带外的消息。它可用于检测服务器不理解的客户端发送的命令。这可能有助于识别配置错误或使用比服务器更近期的 MySQL 版本的客户端，或者试图攻击服务器的客户端。

+   `statement/sql/error`用于处理无法解析的 SQL 语句。它可用于检测客户端发送的格式错误的查询。无法解析的查询与解析但在执行过程中由于错误而失败的查询不同。例如，`SELECT * FROM`是格式错误的，将使用`statement/sql/error`仪器。相比之下，`SELECT *`解析但由于`No tables used`错误而失败。在这种情况下，将使用`statement/sql/select`，并且语句事件包含信息以指示错误的性质。

请求可以从这些来源中获取：

+   作为客户端发送的命令或语句请求，该请求以数据包形式发送

+   作为从副本的中继日志中读取的语句字符串

+   作为事件来自事件调度程序

对于请求的详细信息最初是未知的，性能模式会根据请求的来源从抽象到具体的仪器名称进行顺序处理。

对于从客户端接收的请求：

1.  当服务器在套接字级别检测到新数据包时，将以`statement/abstract/new_packet`的抽象仪器名称开始一个新语句。

1.  当服务器读取数据包编号时，它会更多地了解收到的请求类型，并且性能模式会细化仪器名称。例如，如果请求是一个`COM_PING`数据包，则仪器名称变为`statement/com/Ping`，这是最终名称。如果请求是一个`COM_QUERY`数据包，则已知它对应于一个 SQL 语句，但不知道具体的语句类型。在这种情况下，仪器从一个抽象名称更改为一个更具体但仍然抽象的名称，`statement/abstract/Query`，并且请求需要进一步分类。

1.  如果请求是一个语句，语句文本将被读取并传递给解析器。解析后，确切的语句类型就会知道。例如，如果请求是一个`INSERT`语句，性能模式会将仪器名称从`statement/abstract/Query`细化为`statement/sql/insert`，这是最终名称。

对于从副本的中继日志中读取的请求作为语句：

1.  中继日志中的语句以文本形式存储并按此方式读取。没有网络协议，因此不使用`statement/abstract/new_packet`工具。相反，初始工具是`statement/abstract/relay_log`。

1.  当语句被解析时，确切的语句类型是已知的。例如，如果请求是一个`INSERT`语句，性能模式将从`statement/abstract/Query`细化工具名称为`statement/sql/insert`，这是最终名称。

上述描述仅适用于基于语句的复制。对于基于行的复制，作为处理行更改的副本上的表 I/O 可以被检测，但中继日志中的行事件不会显示为离散语句。

对于从事件调度程序接收的请求：

事件执行使用名称`statement/scheduler/event`进行检测。这是最终名称。

在事件体内执行的语句使用`statement/sql/*`名称进行检测，而不使用任何前置的抽象工具。事件是一个存储过程，并且存储过程在执行之前在内存中预编译。因此，在运行时没有解析，每个语句的类型在执行时已知。

在事件体内执行的语句是子语句。例如，如果一个事件执行了一个`INSERT`语句，那么事件本身的执行是父级，使用`statement/scheduler/event`进行检测，而`INSERT`是子级，使用`statement/sql/insert`进行检测。父子关系存在于*不同*的被检测操作之间。这与在*单个*被检测操作内发生的从抽象到最终工具名称的细化顺序不同。

要收集语句的统计信息，仅启用单个语句类型所使用的最终`statement/sql/*`工具是不够的。还必须启用抽象的`statement/abstract/*`工具。这通常不是问题，因为所有语句工具默认都是启用的。然而，选择性启用或禁用语句工具的应用程序必须考虑到禁用抽象工具也会禁用对单个语句工具的统计信息收集。例如，要收集`INSERT`语句的统计信息，必须启用`statement/sql/insert`，还必须启用`statement/abstract/new_packet`和`statement/abstract/Query`。同样，要对复制语句进行检测，必须启用`statement/abstract/relay_log`。

对于`statement/abstract/Query`等抽象工具，不会对其进行统计汇总，因为最终语句名称中从未将语句分类为抽象工具。
