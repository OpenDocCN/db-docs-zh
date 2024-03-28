> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-events.html`](https://dev.mysql.com/doc/refman/8.0/en/show-events.html)

#### 15.7.7.18 显示事件语句

```sql
SHOW EVENTS
    [{FROM | IN} *schema_name*]
    [LIKE '*pattern*' | WHERE *expr*]
```

此语句显示有关事件管理器事件的信息，这些事件在第 27.4 节“使用事件调度程序”中讨论。它需要对要显示事件的数据库具有`EVENT`权限。

在其最简单形式中，`显示事件`列出当前模式中的所有事件：

```sql
mysql> SELECT CURRENT_USER(), SCHEMA();
+----------------+----------+
| CURRENT_USER() | SCHEMA() |
+----------------+----------+
| jon@ghidora    | myschema |
+----------------+----------+
1 row in set (0.00 sec)

mysql> SHOW EVENTS\G
*************************** 1\. row ***************************
                  Db: myschema
                Name: e_daily
             Definer: jon@ghidora
           Time zone: SYSTEM
                Type: RECURRING
          Execute at: NULL
      Interval value: 1
      Interval field: DAY
              Starts: 2018-08-08 11:06:34
                Ends: NULL
              Status: ENABLED
          Originator: 1
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
```

要查看特定模式的事件，请使用`FROM`子句。例如，要查看`test`模式的事件，请使用以下语句：

```sql
SHOW EVENTS FROM test;
```

如果存在`LIKE`子句，则指示要匹配的事件名称。可以使用`WHERE`子句选择使用更一般条件的行，如第 28.8 节“SHOW 语句的扩展”中讨论的那样。

`显示事件`输出包括以下列：

+   `数据库`

    事件所属的模式（数据库）的名称。

+   `名称`

    事件的名称。

+   `定义者`

    创建事件的用户账户，格式为`'*`user_name`*'@'*`host_name`*`。

+   `时区`

    事件的时区，用于调度事件并在事件执行时生效的时区。默认值为`SYSTEM`。

+   `类型`

    事件重复类型，可以是`ONE TIME`（瞬时）或`RECURRING`（重复）。

+   `执行时间`

    对于一次性事件，这是在`CREATE EVENT`语句的`AT`子句中指定的`DATETIME`值，或者最后一个修改事件的`ALTER EVENT`语句中指定的值。此列中显示的值反映了事件的`AT`子句中包含的任何`INTERVAL`值的加减。例如，如果使用`ON SCHEDULE AT CURRENT_TIMESTAMP + '1:6' DAY_HOUR`创建事件，并且事件在 2018-02-09 14:05:30 创建，则此列中显示的值将是`'2018-02-10 20:05:30'`。如果事件的时间由`EVERY`子句确定而不是`AT`子句（即事件是重复的），则此列的值为`NULL`。

+   `间隔值`

    对于重复事件，事件执行之间等待的间隔数。对于瞬时事件，此列的值始终为`NULL`。

+   `间隔字段`

    用于重复事件等待重复之前的间隔的时间单位。对于瞬时事件，此列的值始终为`NULL`。

+   `开始时间`

    循环事件的开始日期和时间。显示为`DATETIME`值，如果未为事件定义开始日期和时间，则为`NULL`。对于瞬时事件，此列始终为`NULL`。对于定义包含`STARTS`子句的循环事件，此列包含相应的`DATETIME`值。与`Execute At`列一样，此值解析任何使用的表达式。如果没有影响事件定时的`STARTS`子句，则此列为`NULL`。

+   `结束`

    对于定义包含`ENDS`子句的循环事件，此列包含相应的`DATETIME`值。与`Execute At`列一样，此值解析任何使用的表达式。如果没有影响事件定时的`ENDS`子句，则此列为`NULL`。

+   `状态`

    事件状态。`ENABLED`、`DISABLED`或`SLAVESIDE_DISABLED`之一。`SLAVESIDE_DISABLED`表示事件的创建发生在另一个作为复制源的 MySQL 服务器上，并复制到当前作为副本的 MySQL 服务器，但事件目前未在副本上执行。有关更多信息，请参见 Section 19.5.1.16, “Replication of Invoked Features”。

+   `发起者`

    创建事件的 MySQL 服务器的服务器 ID；用于复制。如果在源服务器上执行，则此值可能会被`ALTER EVENT`更新为该语句发生的服务器的服务器 ID。默认值为 0。

+   `character_set_client`

    事件创建时`character_set_client`系统变量的会话值。

+   `collation_connection`

    事件创建时`collation_connection`系统变量的会话值。

+   `数据库排序规则`

    事件关联的数据库的排序规则。

有关`SLAVESIDE_DISABLED`和`发起者`列的更多信息，请参见 Section 19.5.1.16, “Replication of Invoked Features”。

`SHOW EVENTS`显示的时间以事件时区显示，如 Section 27.4.4, “Event Metadata”中所述。

事件信息也可以从`INFORMATION_SCHEMA` `EVENTS`表中获取。请参见 Section 28.3.14, “The INFORMATION_SCHEMA EVENTS Table”。

事件操作语句不会显示在`SHOW EVENTS`的输出中。请使用`SHOW CREATE EVENT`或`INFORMATION_SCHEMA` `EVENTS`表。
