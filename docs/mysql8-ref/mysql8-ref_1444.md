> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-invoked.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-invoked.html)

#### 19.5.1.16 调用特性的复制

调用特性的复制，如可加载函数和存储程序（存储过程和函数、触发器和事件），提供以下特性：

+   特性的效果始终会被复制。

+   以下语句使用基于语句的复制进行复制：

    +   `CREATE EVENT`

    +   `ALTER EVENT`

    +   `DROP EVENT`

    +   `CREATE PROCEDURE`

    +   `DROP PROCEDURE`

    +   `CREATE FUNCTION`

    +   `DROP FUNCTION`

    +   `CREATE TRIGGER`

    +   `DROP TRIGGER`

    但是，使用这些语句创建、修改或删除的特性的*效果*是使用基于行的复制进行复制的。

    注意

    尝试使用基于语句的复制复制调用特性会产生警告“Statement is not safe to log in statement format”。例如，尝试使用基于语句的复制复制可加载函数会生成此警告，因为当前无法由 MySQL 服务器确定函数是否是确定性的。如果您绝对确定调用特性的效果是确定性的，可以安全地忽略此类警告。

+   在`CREATE EVENT`和`ALTER EVENT`的情况下：

    +   事件的状态在副本上设置为`SLAVESIDE_DISABLED`，无论指定的状态如何（这不适用于`DROP EVENT`）。

    +   在副本上，通过其服务器 ID 标识创建事件的源。`INFORMATION_SCHEMA.EVENTS`中的`ORIGINATOR`列存储此信息。有关更多信息，请参见第 15.7.7.18 节，“SHOW EVENTS Statement”。

+   该功能的实现位于副本中，处于可更新状态，因此如果源失败，副本可以被用作源而不会丢失事件处理。

要确定在 MySQL 服务器上是否有任何在不同服务器（作为源服务器）上创建的计划事件，请以类似于此处所示的方式查询信息模式`EVENTS`表：

```sql
SELECT EVENT_SCHEMA, EVENT_NAME
    FROM INFORMATION_SCHEMA.EVENTS
    WHERE STATUS = 'SLAVESIDE_DISABLED';
```

或者，您可以使用`SHOW EVENTS`语句，如下所示：

```sql
SHOW EVENTS
    WHERE STATUS = 'SLAVESIDE_DISABLED';
```

将具有此类事件的副本升级为源时，必须使用`ALTER EVENT *`event_name`* ENABLE`来启用每个事件，其中*`event_name`*是事件的名称。

如果在创建此副本上的事件时涉及多个源，并且您希望识别仅在具有服务器 ID *`source_id`*的给定源上创建的事件，请修改前面在`EVENTS`表上的查询，包括`ORIGINATOR`列，如下所示：

```sql
SELECT EVENT_SCHEMA, EVENT_NAME, ORIGINATOR
    FROM INFORMATION_SCHEMA.EVENTS
    WHERE STATUS = 'SLAVESIDE_DISABLED'
    AND   ORIGINATOR = '*source_id*'
```

您可以以类似的方式使用`SHOW EVENTS`语句中的`ORIGINATOR`：

```sql
SHOW EVENTS
    WHERE STATUS = 'SLAVESIDE_DISABLED'
    AND   ORIGINATOR = '*source_id*'
```

在启用从源复制的事件之前，您应该在副本上禁用 MySQL 事件调度程序（使用类似于`SET GLOBAL event_scheduler = OFF;`的语句），运行任何必要的`ALTER EVENT`语句，重新启动服务器，然后在副本上重新启用事件调度程序（使用类似于`SET GLOBAL event_scheduler = ON;`的语句）-

如果稍后将新源重新降级为副本，则必须手动禁用由`ALTER EVENT`语句启用的所有事件。您可以通过在单独的表中存储先前显示的`SELECT`语句中的事件名称，或使用`ALTER EVENT`语句将事件重命名为具有`replicated_`前缀的公共前缀来执行此操作。

如果您重命名事件，那么在将此服务器重新降级为副本时，您可以通过查询`EVENTS`表来识别事件，如下所示：

```sql
SELECT CONCAT(EVENT_SCHEMA, '.', EVENT_NAME) AS 'Db.Event'
      FROM INFORMATION_SCHEMA.EVENTS
      WHERE INSTR(EVENT_NAME, 'replicated_') = 1;
```
