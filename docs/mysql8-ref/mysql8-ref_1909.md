# [28.3.14 INFORMATION_SCHEMA EVENTS Table](https://dev.mysql.com/doc/refman/8.0/en/information-schema-events-table.html)

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-events-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-events-table.html)

[`EVENTS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-events-table.html)表提供有关事件管理器事件的信息，这些事件在第 27.4 节“使用事件调度程序”中讨论。

[`EVENTS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-events-table.html)表具有以下列：

+   `EVENT_CATALOG`

    事件所属的目录的名称。此值始终为`def`。

+   `EVENT_SCHEMA`

    事件所属的模式（数据库）的名称。

+   `EVENT_NAME`

    事件的名称。

+   `DEFINER`

    `DEFINER`子句中指定的帐户（通常是创建事件的用户），格式为`'*`user_name`*'@'*`host_name`*'`。

+   `TIME_ZONE`

    事件时区，用于调度事件并在事件执行时生效的时区。默认值为`SYSTEM`。

+   `EVENT_BODY`

    事件的`DO`子句中语句的语言。该值始终为`SQL`。

+   `EVENT_DEFINITION`

    事件的`DO`子句组成的 SQL 语句的文本；换句话说，此事件执行的语句。

+   `EVENT_TYPE`

    事件重复类型，可以是`ONE TIME`（瞬时）或`RECURRING`（重复）。

+   `EXECUTE_AT`

    对于一次性事件，这是在`CREATE EVENT`语句的`AT`子句中指定的`DATETIME`值，用于创建事件的最后一个`ALTER EVENT`语句，或修改事件的最后一个语句。此列中显示的值反映了事件的`AT`子句中包含的任何`INTERVAL`值的加法或减法。例如，如果使用`ON SCHEDULE AT CURRENT_TIMESTAMP + '1:6' DAY_HOUR`创建事件，并且事件在 2018-02-09 14:05:30 创建，则此列中显示的值将为`'2018-02-10 20:05:30'`。如果事件的时间由`EVERY`子句确定而不是`AT`子句（即，如果事件是重复的），则此列的值为`NULL`。

+   `INTERVAL_VALUE`

    对于重复事件，事件执行之间等待的间隔数。对于瞬时事件，该值始终为`NULL`。

+   `INTERVAL_FIELD`

    用于重复事件等待重复之前的间隔的时间单位。对于瞬时事件，该值始终为`NULL`。

+   `SQL_MODE`

    创建或更改事件时生效的 SQL 模式，以及事件执行时使用的模式。有关允许的值，请参见第 7.1.11 节“服务器 SQL 模式”。

+   `STARTS`

    重复事件的开始日期和时间。 这显示为 `DATETIME` 值，并且如果未为事件定义开始日期和时间，则为 `NULL`。 对于瞬时事件，此列始终为 `NULL`。 对于定义包含 `STARTS` 子句的重复事件，此列包含相应的 `DATETIME` 值。 与 `EXECUTE_AT` 列一样，此值解析任何使用的表达式。 如果没有 `STARTS` 子句影响事件的时间，此��为 `NULL`。

+   `ENDS`

    对于定义包含 `ENDS` 子句的重复事件，此列包含相应的 `DATETIME` 值。 与 `EXECUTE_AT` 列一样，此值解析任何使用的表达式。 如果没有 `ENDS` 子句影响事件的时间，此列为 `NULL`。

+   `STATUS`

    事件状态。 `ENABLED`、`DISABLED` 或 `SLAVESIDE_DISABLED` 中的一个。 `SLAVESIDE_DISABLED` 表示事件的创建发生在另一个充当复制源的 MySQL 服务器上，并被复制到当前充当副本的 MySQL 服务器，但事件目前未在副本上执行。 有关更多信息，请参见 Section 19.5.1.16, “Replication of Invoked Features”。

+   `ON_COMPLETION`

    两个值之一 `PRESERVE` 或 `NOT PRESERVE`。

+   `CREATED`

    事件创建的日期和时间。 这是一个 `TIMESTAMP` 值。

+   `LAST_ALTERED`

    事件上次修改的日期和时间。 这是一个 `TIMESTAMP` 值。 如果事件自创建以来未被修改，则此值与 `CREATED` 值相同。

+   `LAST_EXECUTED`

    事件上次执行的日期和时间。 这是一个 `DATETIME` 值。 如果事件从未执行过，则此列为 `NULL`。

    `LAST_EXECUTED` 表示事件开始的时间。 因此，`ENDS` 列永远不会小于 `LAST_EXECUTED`。

+   `EVENT_COMMENT`

    如果事件有评论，则为评论的文本。 如果没有，则此值为空。

+   `ORIGINATOR`

    MySQL 服务器的服务器 ID，在该服务器上创建事件时使用；用于复制。 如果在复制源上执行，则此值可能会由 `ALTER EVENT` 更新为发生该语句的服务器的服务器 ID。 默认值为 0。

+   `CHARACTER_SET_CLIENT`

    事件创建时的 `character_set_client` 系统变量的会话值。

+   `COLLATION_CONNECTION`

    事件创建时用户 `'jon'@'ghidora'` 的 `collation_connection` 系统变量的会话值。

+   `DATABASE_COLLATION`

    与事件关联的数据库的排序规则。

#### 注意

+   `EVENTS` 是一个非标准的 `INFORMATION_SCHEMA` 表。

+   `EVENTS` 表中的时间使用事件时区、当前会话时区或 UTC 显示，如 Section 27.4.4, “事件元数据” 中所述。

+   有关 `SLAVESIDE_DISABLED` 和 `ORIGINATOR` 列的更多信息，请参见 Section 19.5.1.16, “调用特性的复制”。

#### 例子

假设用户 `'jon'@'ghidora'` 创建了一个名为 `e_daily` 的事件，然后几分钟后使用 `ALTER EVENT` 语句对其进行修改，如下所示：

```sql
DELIMITER |

CREATE EVENT e_daily
    ON SCHEDULE
      EVERY 1 DAY
    COMMENT 'Saves total number of sessions then clears the table each day'
    DO
      BEGIN
        INSERT INTO site_activity.totals (time, total)
          SELECT CURRENT_TIMESTAMP, COUNT(*)
            FROM site_activity.sessions;
        DELETE FROM site_activity.sessions;
      END |

DELIMITER ;

ALTER EVENT e_daily
    ENABLE;
```

（注意，注释可以跨越多行。）

然后，该用户可以运行以下 `SELECT` 语句，并获得如下输出：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.EVENTS
       WHERE EVENT_NAME = 'e_daily'
       AND EVENT_SCHEMA = 'myschema'\G
*************************** 1\. row ***************************
       EVENT_CATALOG: def
        EVENT_SCHEMA: myschema
          EVENT_NAME: e_daily
             DEFINER: jon@ghidora
           TIME_ZONE: SYSTEM
          EVENT_BODY: SQL
    EVENT_DEFINITION: BEGIN
        INSERT INTO site_activity.totals (time, total)
          SELECT CURRENT_TIMESTAMP, COUNT(*)
            FROM site_activity.sessions;
        DELETE FROM site_activity.sessions;
      END
          EVENT_TYPE: RECURRING
          EXECUTE_AT: NULL
      INTERVAL_VALUE: 1
      INTERVAL_FIELD: DAY
            SQL_MODE: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                      NO_ZERO_IN_DATE,NO_ZERO_DATE,
                      ERROR_FOR_DIVISION_BY_ZERO,
                      NO_ENGINE_SUBSTITUTION
              STARTS: 2018-08-08 11:06:34
                ENDS: NULL
              STATUS: ENABLED
       ON_COMPLETION: NOT PRESERVE
             CREATED: 2018-08-08 11:06:34
        LAST_ALTERED: 2018-08-08 11:06:34
       LAST_EXECUTED: 2018-08-08 16:06:34
       EVENT_COMMENT: Saves total number of sessions then clears the
                      table each day
          ORIGINATOR: 1
CHARACTER_SET_CLIENT: utf8mb4
COLLATION_CONNECTION: utf8mb4_0900_ai_ci
  DATABASE_COLLATION: utf8mb4_0900_ai_ci
```

事件信息也可以通过 `SHOW EVENTS` 语句获取。参见 Section 15.7.7.18, “SHOW EVENTS 语句”。以下语句是等效的：

```sql
SELECT
    EVENT_SCHEMA, EVENT_NAME, DEFINER, TIME_ZONE, EVENT_TYPE, EXECUTE_AT,
    INTERVAL_VALUE, INTERVAL_FIELD, STARTS, ENDS, STATUS, ORIGINATOR,
    CHARACTER_SET_CLIENT, COLLATION_CONNECTION, DATABASE_COLLATION
  FROM INFORMATION_SCHEMA.EVENTS
  WHERE table_schema = '*db_name*'
  [AND column_name LIKE '*wild*']

SHOW EVENTS
  [FROM *db_name*]
  [LIKE '*wild*']
```
