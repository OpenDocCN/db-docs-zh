# 27.4.6 事件调度程序和 MySQL 权限

> 原文：[`dev.mysql.com/doc/refman/8.0/en/events-privileges.html`](https://dev.mysql.com/doc/refman/8.0/en/events-privileges.html)

要启用或禁用计划事件的执行，需要设置全局`event_scheduler`系统变量的值。这需要足够的权限来设置全局系统变量。参见第 7.1.9.1 节，“系统变量权限”。

`EVENT`权限管理事件的创建、修改和删除。可以使用`GRANT`授予此权限。例如，此`GRANT`语句将在用户`jon@ghidora`上为名为`myschema`的模式授予`EVENT`权限：

```sql
GRANT EVENT ON myschema.* TO jon@ghidora;
```

（我们假设该用户帐户已经存在，并且我们希望其保持不变。）

要授予同一用户在所有模式上的`EVENT`权限，请使用以下语句：

```sql
GRANT EVENT ON *.* TO jon@ghidora;
```

`EVENT`权限具有全局或模式级别范围。因此，尝试在单个表上授予它会导致错误，如下所示：

```sql
mysql> GRANT EVENT ON myschema.mytable TO jon@ghidora;
ERROR 1144 (42000): Illegal GRANT/REVOKE command; please
consult the manual to see which privileges can be used
```

重要的是要理解事件是以其定义者的权限执行的，并且它不能执行其定义者没有所需权限的任何操作。例如，假设`jon@ghidora`对`myschema`具有`EVENT`权限。还假设该用户对`myschema`具有`SELECT`权限，但对该模式没有其他权限。`jon@ghidora`可以创建一个新事件，如下所示：

```sql
CREATE EVENT e_store_ts
    ON SCHEDULE
      EVERY 10 SECOND
    DO
      INSERT INTO myschema.mytable VALUES (UNIX_TIMESTAMP());
```

用户等待一分钟左右，然后执行`SELECT * FROM mytable;`查询，期望在表中看到几行新数据。然而，表是空的。由于用户没有表的`INSERT`权限，因此事件没有效果。

如果检查 MySQL 错误日志（`*`hostname`*.err`），您会看到事件正在执行，但它尝试执行的操作失败：

```sql
2013-09-24T12:41:31.261992Z 25 [ERROR] Event Scheduler:
[jon@ghidora][cookbook.e_store_ts] INSERT command denied to user
'jon'@'ghidora' for table 'mytable'
2013-09-24T12:41:31.262022Z 25 [Note] Event Scheduler:
[jon@ghidora].[myschema.e_store_ts] event execution failed.
2013-09-24T12:41:41.271796Z 26 [ERROR] Event Scheduler:
[jon@ghidora][cookbook.e_store_ts] INSERT command denied to user
'jon'@'ghidora' for table 'mytable'
2013-09-24T12:41:41.272761Z 26 [Note] Event Scheduler:
[jon@ghidora].[myschema.e_store_ts] event execution failed.
```

由于这个用户很可能无法访问错误日志，可以通过直接执行事件的操作语句来验证该事件是否有效：

```sql
mysql> INSERT INTO myschema.mytable VALUES (UNIX_TIMESTAMP());
ERROR 1142 (42000): INSERT command denied to user
'jon'@'ghidora' for table 'mytable'
```

检查信息模式`EVENTS`表显示`e_store_ts`存在且已启用，但其`LAST_EXECUTED`列为`NULL`：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.EVENTS
     >     WHERE EVENT_NAME='e_store_ts'
     >     AND EVENT_SCHEMA='myschema'\G
*************************** 1\. row ***************************
   EVENT_CATALOG: NULL
    EVENT_SCHEMA: myschema
      EVENT_NAME: e_store_ts
         DEFINER: jon@ghidora
      EVENT_BODY: SQL
EVENT_DEFINITION: INSERT INTO myschema.mytable VALUES (UNIX_TIMESTAMP())
      EVENT_TYPE: RECURRING
      EXECUTE_AT: NULL
  INTERVAL_VALUE: 5
  INTERVAL_FIELD: SECOND
        SQL_MODE: NULL
          STARTS: 0000-00-00 00:00:00
            ENDS: 0000-00-00 00:00:00
          STATUS: ENABLED
   ON_COMPLETION: NOT PRESERVE
         CREATED: 2006-02-09 22:36:06
    LAST_ALTERED: 2006-02-09 22:36:06
   LAST_EXECUTED: NULL
   EVENT_COMMENT: 1 row in set (0.00 sec)
```

要撤销`EVENT`权限，请使用`REVOKE`语句。在此示例中，从`jon@ghidora`用户账户中移除了模式`myschema`上的`EVENT`权限：

```sql
REVOKE EVENT ON myschema.* FROM jon@ghidora;
```

重要提示

从用户那里撤销`EVENT`权限不会删除或禁用该用户可能创建的任何事件。

事件不会因为创建它的用户被重命名或删除而迁移或删除。

假设用户`jon@ghidora`已被授予`myschema`模式上的`EVENT`和`INSERT`权限。然后该用户创建了以下事件：

```sql
CREATE EVENT e_insert
    ON SCHEDULE
      EVERY 7 SECOND
    DO
      INSERT INTO myschema.mytable;
```

创建了这个事件后，`root`撤销了`jon@ghidora`的`EVENT`权限。然而，`e_insert`继续执行，每七秒插入一行到`mytable`中。如果`root`发出了以下任一语句，情况也是如此：

+   `DROP USER jon@ghidora;`

+   `RENAME USER jon@ghidora TO someotherguy@ghidora;`

您可以通过在发出`DROP USER`或`RENAME USER`语句之前和之后检查信息模式`EVENTS`表来验证这一点。

事件定义存储在数据字典中。要删除由另一个用户账户创建的事件，您必须是 MySQL `root`用户或具有必要权限的其他用户。

用户的`EVENT`权限存储在`mysql.user`和`mysql.db`表的`Event_priv`列中。在这两种情况下，该列保存值'`Y`'或'`N`'中的一个。'`N`'是默认值。对于给定用户，只有当该用户具有全局`EVENT`权限（即，如果使用`GRANT EVENT ON *.*`授予了权限）时，`mysql.user.Event_priv`才设置为'`Y`'。对于模式级别的`EVENT`权限，`GRANT`在`mysql.db`中创建一行，并将该行的`Db`列设置为模式名称，`User`列设置为用户名称，`Event_priv`列设置为'`Y`'。永远不应该直接操作这些表，因为`GRANT EVENT`和`REVOKE EVENT`语句会在其上执行所需的操作。

五个状态变量提供了与事件相关操作的计数（但*不*包括事件执行的语句；请参阅第 27.8 节，“存储程序的限制”）。这些是：

+   `Com_create_event`: 自上次服务器重启以来执行的`CREATE EVENT`语句的次数。

+   `Com_alter_event`: 自上次服务器重启以来执行的`ALTER EVENT`语句的次数。

+   `Com_drop_event`: 自上次服务器重启以来执行的`DROP EVENT`语句的次数。

+   `Com_show_create_event`: 自上次服务器重启以来执行的`SHOW CREATE EVENT`语句的次数。

+   `Com_show_events`: 自上次服务器重启以来执行的`SHOW EVENTS`语句的次数。

通过运行语句`SHOW STATUS LIKE '%event%';`，您可以一次查看所有这些的当前值。
