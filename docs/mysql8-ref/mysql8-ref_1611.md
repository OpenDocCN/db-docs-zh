# 22.5.7 监控 X 插件

> 译文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-system-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-system-monitoring.html)

对于一般的 X 插件监控，请使用其公开的状态变量。参见第 22.5.6.3 节，“X 插件状态变量”。有关专门监视消息压缩效果的信息，请参见 X 插件的连接压缩监控。

#### 监控 X 插件生成的 SQL

本节描述了如何监视运行 X DevAPI 操作时 X 插件生成的 SQL 语句。当您执行 CRUD 语句时，它会被转换为 SQL 并针对服务器执行。要能够监视生成的 SQL，必须启用 Performance Schema 表。SQL 注册在`performance_schema.events_statements_current`、`performance_schema.events_statements_history`和`performance_schema.events_statements_history_long`表下。以下示例使用了在本节快速入门教程中导入的`world_x`模式。我们在 Python 模式下使用 MySQL Shell，并使用`\sql`命令，该命令使您能够发出 SQL 语句而无需切换到 SQL 模式。这很重要，因为如果您尝试切换到 SQL 模式，该过程将显示此操作的结果而不是 X DevAPI 操作。如果您使用 JavaScript 模式下的 MySQL Shell，则`\sql`命令的使用方式相同。

1.  检查`events_statements_history`消费者是否已启用。问题：

    ```sql
    mysql-py> \sql SELECT enabled FROM performance_schema.setup_consumers WHERE NAME = 'events_statements_history'
    +---------+
    | enabled |
    +---------+
    | YES     |
    +---------+
    ```

1.  检查所有仪器是否向消费者报告数据。问题：

    ```sql
    mysql-py> \sql SELECT NAME, ENABLED, TIMED FROM performance_schema.setup_instruments WHERE NAME LIKE 'statement/%' AND NOT (ENABLED and TIMED)
    ```

    如果此语句报告至少一行，则需要启用仪器。请参见第 29.4 节，“Performance Schema 运行时配置”。

1.  获取当前连接的线程 ID。问题：

    ```sql
    mysql-py> \sql SELECT thread_id INTO @id FROM performance_schema.threads WHERE processlist_id=connection_id()
    ```

1.  执行您想要查看生成的 SQL 的 X DevAPI CRUD 操作。例如，问题：

    ```sql
    mysql-py> db.CountryInfo.find("Name = :country").bind("country", "Italy")
    ```

    您必须不再发出任何操作，以便下一步显示正确的结果。

1.  显示由此线程 ID 执行的最后一个 SQL 查询。问题：

    ```sql
    mysql-py> \sql SELECT THREAD_ID, MYSQL_ERRNO,SQL_TEXT FROM performance_schema.events_statements_history WHERE THREAD_ID=@id ORDER BY TIMER_START DESC LIMIT 1;
    +-----------+-------------+--------------------------------------------------------------------------------------+
    | THREAD_ID | MYSQL_ERRNO | SQL_TEXT                                                                             |
    +-----------+-------------+--------------------------------------------------------------------------------------+
    |        29 |           0 | SELECT doc FROM `world_x`.`CountryInfo` WHERE (JSON_EXTRACT(doc,'$.Name') = 'Italy') |
    +-----------+-------------+--------------------------------------------------------------------------------------+
    ```

    结果显示了基于最近语句生成的 X 插件 SQL，本例中为上一步的 X DevAPI CRUD 操作。
