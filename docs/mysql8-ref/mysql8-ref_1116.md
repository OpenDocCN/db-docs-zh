> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-triggers.html`](https://dev.mysql.com/doc/refman/8.0/en/show-triggers.html)

#### 15.7.7.40 SHOW TRIGGERS Statement

```sql
SHOW TRIGGERS
    [{FROM | IN} *db_name*]
    [LIKE '*pattern*' | WHERE *expr*]
```

`SHOW TRIGGERS`列出了当前为数据库中的表定义的触发器（默认数据库，除非给出`FROM`子句）。此语句仅对具有`TRIGGER`权限的数据库和表返回结果。如果存在`LIKE`子句，则指示匹配哪些表名（而不是触发器名称）并导致语句显示这些表的触发器。可以使用`WHERE`子句来选择使用更一般条件选择行，如第 28.8 节，“SHOW 语句的扩展”中讨论的那样。

对于在第 27.3 节，“使用触发器”中定义的`ins_sum`触发器，`SHOW TRIGGERS`的输出如下所示：

```sql
mysql> SHOW TRIGGERS LIKE 'acc%'\G
*************************** 1\. row ***************************
             Trigger: ins_sum
               Event: INSERT
               Table: account
           Statement: SET @sum = @sum + NEW.amount
              Timing: BEFORE
             Created: 2018-08-08 10:10:12.61
            sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                      NO_ZERO_IN_DATE,NO_ZERO_DATE,
                      ERROR_FOR_DIVISION_BY_ZERO,
                      NO_ENGINE_SUBSTITUTION
             Definer: me@localhost
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
```

`SHOW TRIGGERS`输出具有以下列：

+   `触发器`

    触发器的名称。

+   `事件`

    触发事件。这是触发器激活的相关表上的操作类型。值为`INSERT`（插入了一行），`DELETE`（删除了一行）或`UPDATE`（修改了一行）。

+   `表`

    定义触发器的表。

+   `语句`

    触发器主体；即触发器激活时执行的语句。

+   `时机`

    触发器在触发事件之前还是之后激活。值为`BEFORE`或`AFTER`。

+   `创建���间`

    触发器创建的日期和时间。这是一个`TIMESTAMP(2)`值（带有百分之一秒的小数部分）。

+   `sql_mode`

    触发器创建时生效的 SQL 模式，以及触发器执行的模式。有关允许的值，请参见第 7.1.11 节，“服务器 SQL 模式”。

+   `定义者`

    创建触发器的用户的帐户，格式为`'*`user_name`*'@'*`host_name`*'`。

+   `character_set_client`

    触发器创建时的`character_set_client`系统变量的会话值。

+   `collation_connection`

    触发器创建时的`collation_connection`系统变量的会话值。

+   `数据库排序规则`

    触发器关联的数据库的排序规则。

触发器信息也可以从`INFORMATION_SCHEMA` `TRIGGERS`表中获取。请参阅第 28.3.45 节，“INFORMATION_SCHEMA TRIGGERS 表”。
