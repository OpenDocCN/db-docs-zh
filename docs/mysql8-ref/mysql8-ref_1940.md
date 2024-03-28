# 28.3.45 信息模式 TRIGGERS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-triggers-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-triggers-table.html)

`TRIGGERS`表提供有关触发器的信息。要查看有关表格触发器的信息，您必须对表格具有`TRIGGER`权限。

`TRIGGERS`表具有以下列：

+   `TRIGGER_CATALOG`

    触发器所属的目录名称。此值始终为`def`。

+   `TRIGGER_SCHEMA`

    触发器所属的模式（数据库）的名称。

+   `TRIGGER_NAME`

    触发器的名称。

+   `EVENT_MANIPULATION`

    触发器事件。这是触发器激活的相关表格上的操作类型。值为`INSERT`（插入了一行），`DELETE`（删除了一行）或`UPDATE`（修改了一行）。

+   `EVENT_OBJECT_CATALOG`，`EVENT_OBJECT_SCHEMA`和`EVENT_OBJECT_TABLE`

    如第 27.3 节，“使用触发器”中所述，每个触发器都与一个表格关联。这些列指示此表格所在的目录和模式（数据库），以及表格名称。`EVENT_OBJECT_CATALOG`值始终为`def`。

+   `ACTION_ORDER`

    触发器动作在相同表格上具有相同`EVENT_MANIPULATION`和`ACTION_TIMING`值的触发器列表中的序数位置。

+   `ACTION_CONDITION`

    此值始终为`NULL`。

+   `ACTION_STATEMENT`

    触发器主体；即触发器激活时执行的语句。此文本使用 UTF-8 编码。

+   `ACTION_ORIENTATION`

    此值始终为`ROW`。

+   `ACTION_TIMING`

    触发器在触发事件之前或之后激活。值为`BEFORE`或`AFTER`。

+   `ACTION_REFERENCE_OLD_TABLE`

    此值始终为`NULL`。

+   `ACTION_REFERENCE_NEW_TABLE`

    此值始终为`NULL`。

+   `ACTION_REFERENCE_OLD_ROW`和`ACTION_REFERENCE_NEW_ROW`

    旧列标识符和新列标识符。`ACTION_REFERENCE_OLD_ROW`值始终为`OLD`，`ACTION_REFERENCE_NEW_ROW`值始终为`NEW`。

+   `CREATED`

    触发器创建时的日期和时间。这是一个`TIMESTAMP(2)`值（带有百分之一秒的小数部分）。

+   `SQL_MODE`

    触发器创建时生效的 SQL 模式，以及触发器执行的模式。有关允许的值，请参见第 7.1.11 节，“服务器 SQL 模式”。

+   `DEFINER`

    `DEFINER`子句中命名的帐户（通常是创建触发器的用户），格式为`'*`user_name`*'@'*`host_name`*'`。

+   `CHARACTER_SET_CLIENT`

    触发器创建时的`character_set_client`系统变量的会话值。

+   `COLLATION_CONNECTION`

    触发器创建时`collation_connection`系统变量的会话值。

+   `DATABASE_COLLATION`

    触发器关联的数据库的排序规则。

#### 示例

以下示例使用了 Section 27.3, “Using Triggers”中定义的`ins_sum`触发器：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.TRIGGERS
       WHERE TRIGGER_SCHEMA='test' AND TRIGGER_NAME='ins_sum'\G
*************************** 1\. row ***************************
           TRIGGER_CATALOG: def
            TRIGGER_SCHEMA: test
              TRIGGER_NAME: ins_sum
        EVENT_MANIPULATION: INSERT
      EVENT_OBJECT_CATALOG: def
       EVENT_OBJECT_SCHEMA: test
        EVENT_OBJECT_TABLE: account
              ACTION_ORDER: 1
          ACTION_CONDITION: NULL
          ACTION_STATEMENT: SET @sum = @sum + NEW.amount
        ACTION_ORIENTATION: ROW
             ACTION_TIMING: BEFORE
ACTION_REFERENCE_OLD_TABLE: NULL
ACTION_REFERENCE_NEW_TABLE: NULL
  ACTION_REFERENCE_OLD_ROW: OLD
  ACTION_REFERENCE_NEW_ROW: NEW
                   CREATED: 2018-08-08 10:10:12.61
                  SQL_MODE: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                            NO_ZERO_IN_DATE,NO_ZERO_DATE,
                            ERROR_FOR_DIVISION_BY_ZERO,
                            NO_ENGINE_SUBSTITUTION
                   DEFINER: me@localhost
      CHARACTER_SET_CLIENT: utf8mb4
      COLLATION_CONNECTION: utf8mb4_0900_ai_ci
        DATABASE_COLLATION: utf8mb4_0900_ai_ci
```

触发器信息也可以通过`SHOW TRIGGERS`语句获取。请参阅 Section 15.7.7.40, “SHOW TRIGGERS Statement”。
