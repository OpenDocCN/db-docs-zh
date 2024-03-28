> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-trigger.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-trigger.html)

#### 15.7.7.11 显示 CREATE TRIGGER 语句

```sql
SHOW CREATE TRIGGER *trigger_name*
```

此语句显示创建指定触发器的`CREATE TRIGGER`语句。此语句需要与触发器关联的表的`TRIGGER`权限。

```sql
mysql> SHOW CREATE TRIGGER ins_sum\G
*************************** 1\. row ***************************
               Trigger: ins_sum
              sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                        NO_ZERO_IN_DATE,NO_ZERO_DATE,
                        ERROR_FOR_DIVISION_BY_ZERO,
                        NO_ENGINE_SUBSTITUTION
SQL Original Statement: CREATE DEFINER=`me`@`localhost` TRIGGER `ins_sum`
                        BEFORE INSERT ON `account`
                        FOR EACH ROW SET @sum = @sum + NEW.amount
  character_set_client: utf8mb4
  collation_connection: utf8mb4_0900_ai_ci
    Database Collation: utf8mb4_0900_ai_ci
               Created: 2018-08-08 10:10:12.61
```

`显示 CREATE TRIGGER`输出包含以下列：

+   `触发器`：触发器名称。

+   `sql_mode`：触发器执行时有效的 SQL 模式。

+   `SQL 原始语句`：定义触发器的`CREATE TRIGGER`语句。

+   `character_set_client`：创建触发器时`character_set_client`系统变量的会话值。

+   `collation_connection`：创建触发器时`collation_connection`系统变量的会话值。

+   `数据库排序规则`：与触发器关联的数据库的排序规则。

+   `创建时间`：创建触发器的日期和时间。这是一个`TIMESTAMP(2)`值（带有百分之一秒的小数部分）。

触发器信息也可以从`INFORMATION_SCHEMA` `TRIGGERS`表中获取。请参阅第 28.3.45 节，“INFORMATION_SCHEMA TRIGGERS 表”。
