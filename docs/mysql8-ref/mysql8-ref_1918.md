# 28.3.23 信息模式 PROCESSLIST 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-processlist-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-processlist-table.html)

重要

`INFORMATION_SCHEMA.PROCESSLIST` 已弃用，并将在未来的 MySQL 版本中移除。因此，使用此表的 `SHOW PROCESSLIST` 的实现也已弃用。建议改用性能模式实现的 `PROCESSLIST`。

MySQL 进程列表显示当前在服务器内执行的线程集合正在执行的操作。`PROCESSLIST` 表是进程信息的一个来源。要比较此表与其他来源，请参见 进程信息来源。

`PROCESSLIST` 表具有以下列：

+   `ID`

    连接标识符。这是在 `SHOW PROCESSLIST` 语句中显示的相同值，在性能模式 `threads` 表中的 `PROCESSLIST_ID` 列中显示，并在线程内部由 `CONNECTION_ID()` 函数返回。

+   `USER`

    发出语句的 MySQL 用户。`system user` 的值指代服务器生成的非客户端线程，用于内部处理任务，例如，延迟行处理程序线程或在副本主机上使用的 I/O 或 SQL 线程。对于 `system user`，在 `Host` 列中没有指定主机。`unauthenticated user` 指代已与客户端连接关联但尚未进行客户端用户认证的线程。`event_scheduler` 指代监视计划事件的线程（参见 第 27.4 节，“使用事件调度程序”）。

    注意

    `USER` 值为 `system user` 与 `SYSTEM_USER` 权限是不同的。前者指定内部线程。后者区分系统用户和常规用户账户类别（参见 第 8.2.11 节，“账户类别”）。

+   `HOST`

    发出语句的客户端的主机名（对于`system user`，没有主机）。TCP/IP 连接的主机名以`*`host_name`*:*`client_port`*`的格式报告，以便更容易确定哪个客户端在执行什么操作。

+   `DB`

    线程的默认数据库，如果没有选择任何数据库则为`NULL`。

+   `COMMAND`

    线程代表客户端执行的命令类型，如果会话空闲则为`Sleep`。有关线程命令的描述，请参见 Section 10.14, “Examining Server Thread (Process) Information” Information")。此列的值对应于客户端/服务器协议的`COM_*`xxx`*`命令和`Com_*`xxx`*`状态变量。请参见 Section 7.1.10, “Server Status Variables”。

+   `TIME`

    线程在当前状态下已经持续的秒数。对于复制 SQL 线程，该值是最后一个复制事件的时间戳与复制主机的实际时间之间的秒数。请参见 Section 19.2.3, “Replication Threads”。

+   `STATE`

    表示线程正在执行的操作、事件或状态。有关`STATE`值的描述，请参见 Section 10.14, “Examining Server Thread (Process) Information” Information")。

    大多数状态对应于非常快速的操作。如果一个线程在给定状态停留了很多秒，可能存在需要调查的问题。

+   `INFO`

    线程正在执行的语句，如果没有执行语句则为`NULL`。该语句可能是发送到服务器的语句，或者如果语句执行其他语句，则是最内层的语句。例如，如果一个`CALL`语句执行一个正在执行`SELECT`语句的存储过程，`INFO`值显示`SELECT`语句。

#### 注释

+   `PROCESSLIST`是一个非标准的`INFORMATION_SCHEMA`表。

+   类似于`SHOW PROCESSLIST`语句的输出，`PROCESSLIST`表提供了关于所有线程的信息，即使是属于其他用户的线程，如果你拥有`PROCESS`权限。否则（没有`PROCESS`权限），非匿名用户可以访问自己线程的信息，但不能访问其他用户的线程，匿名用户无法访问线程信息。

+   如果一个 SQL 语句引用了`PROCESSLIST`表，MySQL 会在语句执行开始时一次性填充整个表，因此在语句执行期间存在读一致性。对于多语句事务，不存在读一致性。

以下语句是等价的：

```sql
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST

SHOW FULL PROCESSLIST
```
