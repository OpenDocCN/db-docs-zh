# 27.4.2 事件调度器配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/events-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/events-configuration.html)

事件由一个特殊的事件调度器线程执行；当我们提到事件调度器时，实际上是指这个线程。当运行时，具有`PROCESS`权限的用户可以在`SHOW PROCESSLIST`的输出中看到事件调度器线程及其当前状态，如下面的讨论所示。

全局`event_scheduler`系统变量确定事件调度器在服务器上是否启用和运行。它具有以下值之一，这些值会影响事件调度的描述：

+   `ON`: 事件调度器已启动；事件调度器线程运行并执行所有预定事件。`ON`是默认的`event_scheduler`值。

    当事件调度器为`ON`时，事件调度器线程将作为守护进程在`SHOW PROCESSLIST`的输出中列出，并且其状态如下所示：

    ```sql
    mysql> SHOW PROCESSLIST\G
    *************************** 1\. row ***************************
         Id: 1
       User: root
       Host: localhost
         db: NULL
    Command: Query
       Time: 0
      State: NULL
       Info: show processlist
    *************************** 2\. row ***************************
         Id: 2
       User: event_scheduler
       Host: localhost
         db: NULL
    Command: Daemon
       Time: 3
      State: Waiting for next activation
       Info: NULL 2 rows in set (0.00 sec)
    ```

    通过将`event_scheduler`的值设置为`OFF`来停止事件调度。

+   `OFF`: 事件调度器已停止。事件调度器线程不运行，不会显示在`SHOW PROCESSLIST`的输出中，也不会执行任何预定事件。

    当事件调度器停止时（`event_scheduler`为`OFF`），可以通过将`event_scheduler`的值设置为`ON`来启动它。（见下一项。）

+   `DISABLED`: 此值使事件调度器无法运行。当事件调度器为`DISABLED`时，事件调度器线程不运行（因此不会出现在`SHOW PROCESSLIST`的输出中）。此外，事件调度器状���无法在运行时更改。

如果事件调度器状态未设置为`DISABLED`，则可以在`event_scheduler`的值之间切换为`ON`和`OFF`（使用`SET`）。在设置此变量时，也可以使用`0`表示`OFF`，使用`1`表示`ON`。因此，在**mysql**客户端中可以使用以下任何一条语句来启动事件调度器：

```sql
SET GLOBAL event_scheduler = ON;
SET @@GLOBAL.event_scheduler = ON;
SET GLOBAL event_scheduler = 1;
SET @@GLOBAL.event_scheduler = 1;
```

类似地，可以使用以下任何一条语句关闭事件调度器：

```sql
SET GLOBAL event_scheduler = OFF;
SET @@GLOBAL.event_scheduler = OFF;
SET GLOBAL event_scheduler = 0;
SET @@GLOBAL.event_scheduler = 0;
```

注意

如果事件调度器已启用，则启用`super_read_only`系统变量会阻止其在`events`数据字典表中更新事件“上次执行”时间戳。这会导致事件调度器在下次尝试执行计划事件时停止，并在写入服务器错误日志后发出消息。（在这种情况下，`event_scheduler`系统变量不会从`ON`更改为`OFF`。一个含义是，此变量拒绝了 DBA *意图*，即事件调度器启用或禁用，其实际状态为启动或停止可能是不同的。）如果在启用后随后禁用`super_read_only`，服务器会根据需要自动重新启动事件调度器，截至 MySQL 8.0.26。在 MySQL 8.0.26 之前，需要手动重新启动事件调度器以再次启用它。

尽管`ON`和`OFF`有数值等价物，但由`SELECT`或`SHOW VARIABLES`显示的`event_scheduler`的值始终为`OFF`、`ON`或`DISABLED`。*`DISABLED`没有数值等价物*。因此，在设置此变量时，通常优先选择`ON`和`OFF`而不是`1`和`0`。

请注意，尝试设置`event_scheduler`而不将其指定为全局变量会导致错误：

```sql
mysql< SET @@event_scheduler = OFF;
ERROR 1229 (HY000): Variable 'event_scheduler' is a GLOBAL
variable and should be set with SET GLOBAL
```

重要

只能在服务器启动时将事件调度器设置为`DISABLED`。如果`event_scheduler`为`ON`或`OFF`，则无法在运行时将其设置为`DISABLED`。此外，如果事件调度器在启动时设置为`DISABLED`，则无法在运行时更改`event_scheduler`的值。

要禁用事件调度器，请使用以下两种方法之一：

+   作为启动服务器时的命令行选项：

    ```sql
    --event-scheduler=DISABLED
    ```

+   在服务器配置文件（`my.cnf`，或 Windows 系统上的`my.ini`）中，包含可以被服务器读取的行（例如，在`[mysqld]`部分）：

    ```sql
    event_scheduler=DISABLED
    ```

要启用事件调度器，请在重新启动服务器时不使用`--event-scheduler=DISABLED`命令行选项，或在服务器配置文件中删除或注释包含`event-scheduler=DISABLED`的行，或者在适当的情况下。另外，当启动服务器时，您可以使用`ON`（或`1`）或`OFF`（或`0`）来代替`DISABLED`值。

注意

当`event_scheduler`设置为`DISABLED`时，可以发出事件操作语句。在这种情况下不会生成警告或错误（前提是语句本身有效）。但是，在将此变量设置为`ON`（或`1`）之前，计划事件无法执行。一旦完成此操作，事件调度程序线程将执行所有满足调度条件的事件。

使用`--skip-grant-tables`选项启动 MySQL 服务器会将`event_scheduler`设置为`DISABLED`，覆盖在命令行或`my.cnf`或`my.ini`文件中设置的任何其他值（Bug #26807）。

有关用于创建、修改和删除事件的 SQL 语句，请参见第 27.4.3 节，“事件语法”。

MySQL 在`INFORMATION_SCHEMA`数据库中提供了一个`EVENTS`表。可以查询此表以获取关于服务器上已定义的计划事件的信息。更多信息请参见第 27.4.4 节，“事件元数据”和第 28.3.14 节，“INFORMATION_SCHEMA EVENTS 表”。

有关事件调度和 MySQL 权限系统的信息，请参见第 27.4.6 节，“事件调度程序和 MySQL 权限”。
