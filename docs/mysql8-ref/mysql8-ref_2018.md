> [`dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-threads-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-threads-table.html)

#### 29.12.2.5 The setup_threads Table

`setup_threads` 表列出了被检测的线程类。它公开了线程类的名称和属性：

```sql
mysql> SELECT * FROM performance_schema.setup_threads\G
*************************** 1\. row ***************************
         NAME: thread/performance_schema/setup
      ENABLED: YES
      HISTORY: YES
   PROPERTIES: singleton
   VOLATILITY: 0
DOCUMENTATION: NULL
...
*************************** 4\. row ***************************
         NAME: thread/sql/main
      ENABLED: YES
      HISTORY: YES
   PROPERTIES: singleton
   VOLATILITY: 0
DOCUMENTATION: NULL
*************************** 5\. row ***************************
         NAME: thread/sql/one_connection
      ENABLED: YES
      HISTORY: YES
   PROPERTIES: user
   VOLATILITY: 0
DOCUMENTATION: NULL
...
*************************** 10\. row ***************************
         NAME: thread/sql/event_scheduler
      ENABLED: YES
      HISTORY: YES
   PROPERTIES: singleton
   VOLATILITY: 0
DOCUMENTATION: NULL
```

`setup_threads` 表具有以下列：

+   `NAME`

    仪器名称。线程仪器以`thread`开头（例如，`thread/sql/parser_service`或`thread/performance_schema/setup`）。

+   `ENABLED`

    仪器是否启用。值为`YES`或`NO`。此列可以修改，尽管对于已经运行的线程设置`ENABLED`没有效果。

    对于后台线程，设置`ENABLED`值控制随后为此仪器创建的线程是否在`threads` 表中列出时将`INSTRUMENTED`设置为`YES`或`NO`。对于前台线程，此列没有效果；`setup_actors` 表优先。

+   `HISTORY`

    是否记录仪器的历史事件。值为`YES`或`NO`。此列可以修改，尽管对于已经运行的线程设置`HISTORY`没有效果。

    对于后台线程，设置`HISTORY`值控制随后为此仪器创建的线程是否在`threads` 表中列出时将`HISTORY`设置为`YES`或`NO`。对于前台线程，此列没有效果；`setup_actors` 表优先。

+   `PROPERTIES`

    仪器属性。此列使用`SET`数据类型，因此可以为每个仪器设置以下列表中的多个标志：

    +   `singleton`：该仪器具有单个实例。例如，`thread/sql/main`仪器只有一个线程。

    +   `user`：该仪器与用户工作负载直接相关（而不是系统工作负载）。例如，执行用户会话的线程（如`thread/sql/one_connection`）具有`user`属性，以区分它们与系统线程。

+   `VOLATILITY`

    仪器的波动性。此列与`setup_instruments` 表中的含义相同。请参阅 Section 29.12.2.3, “The setup_instruments Table”。

+   `DOCUMENTATION`

    描述仪器目的的字符串。如果没有可用的描述，则值为`NULL`。

`setup_threads`表具有以下索引：

+   主键为(`NAME`)

不允许对`setup_threads`表执行`TRUNCATE TABLE`操作。
