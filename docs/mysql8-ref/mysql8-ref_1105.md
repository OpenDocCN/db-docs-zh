> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-processlist.html`](https://dev.mysql.com/doc/refman/8.0/en/show-processlist.html)

#### 15.7.7.29 SHOW PROCESSLIST Statement

```sql
SHOW [FULL] PROCESSLIST
```

重要提示

`SHOW PROCESSLIST`的 INFORMATION SCHEMA 实现已被弃用，并将在未来的 MySQL 版本中移除。建议改用 Performance Schema 实现的`SHOW PROCESSLIST`。

MySQL 进程列表显示当前在服务器内执行的线程集合正在执行的操作。`SHOW PROCESSLIST`语句是进程信息的一个来源。有关此语句与其他来源的比较，请参见进程信息的来源。

注意

截至 MySQL 8.0.22，基于 Performance Schema `processlist`表的`SHOW PROCESSLIST`的替代实现已经可用，与默认的`SHOW PROCESSLIST`实现不同，它不需要互斥锁，并具有更好的性能特性。详情请参见第 29.12.21.7 节，“The processlist Table”。

如果你拥有`PROCESS`权限，你可以查看所有线程，甚至属于其他用户的线程。否则（没有`PROCESS`权限），非匿名用户可以访问有关自己线程的信息，但不能访问其他用户的线程，匿名用户无法访问线程信息。

没有`FULL`关键字，`SHOW PROCESSLIST`仅显示`Info`字段中每个语句的前 100 个字符。

如果收到“连接过多”错误消息并想找出原因，`SHOW PROCESSLIST`语句非常有用。MySQL 保留一个额外的连接供具有`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）的帐户使用，以确保管理员始终能够连接并检查系统（假设您没有将此权限授予所有用户）。

线程可以使用`KILL`语句终止。请参见第 15.7.8.4 节，“KILL Statement”。

`SHOW PROCESSLIST`输出示例：

```sql
mysql> SHOW FULL PROCESSLIST\G
*************************** 1\. row ***************************
     Id: 1
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 1030455
  State: Waiting for source to send event
   Info: NULL
*************************** 2\. row ***************************
     Id: 2
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 1004
  State: Has read all relay log; waiting for the replica
         I/O thread to update it
   Info: NULL
*************************** 3\. row ***************************
     Id: 3112
   User: replikator
   Host: artemis:2204
     db: NULL
Command: Binlog Dump
   Time: 2144
  State: Has sent all binlog to replica; waiting for binlog to be updated
   Info: NULL
*************************** 4\. row ***************************
     Id: 3113
   User: replikator
   Host: iconnect2:45781
     db: NULL
Command: Binlog Dump
   Time: 2086
  State: Has sent all binlog to replica; waiting for binlog to be updated
   Info: NULL
*************************** 5\. row ***************************
     Id: 3123
   User: stefan
   Host: localhost
     db: apollon
Command: Query
   Time: 0
  State: NULL
   Info: SHOW FULL PROCESSLIST
```

`SHOW PROCESSLIST`输出具有以下列：

+   `Id`

    连接标识符。这是在`INFORMATION_SCHEMA` `PROCESSLIST`表中的`ID`列中显示的相同值，在性能模式`threads`表中的`PROCESSLIST_ID`列中显示的值，并在线程内部由`CONNECTION_ID()`函数返回。

+   `用户`

    发出语句的 MySQL 用户。`系统用户`的值指的是服务器生成的非客户端线程，用于内部处理任务，例如延迟行处理程序线程或在副本主机上使用的 I/O（接收器）或 SQL（应用程序）线程。对于`系统用户`，在`Host`列中没有指定主机。`未经身份验证的用户`指的是已与客户端连接关联但尚未对客户端用户进行身份验证的线程。`event_scheduler`指的是监视计划事件的线程（参见第 27.4 节，“使用事件调度程序”）。

    注意

    `系统用户`的`用户`值与`SYSTEM_USER`权限是不同的。前者指定内部线程。后者区分系统用户和常规用户账户类别（参见第 8.2.11 节，“账户类别”）。

+   `主机`

    发出语句的客户端的主机名（除了`系统用户`外，没有主机）。TCP/IP 连接的主机名以`*`host_name`*:*`client_port`*`格式报告，以便更容易确定哪个客户端正在执行什么操作。

+   `数据库`

    线程的默认数据库，如果没有选择任何数据库���则为`NULL`。

+   `命令`

    线程代表客户端执行的命令类型，或者如果会话空闲，则为`Sleep`。有关线程命令的描述，请参见第 10.14 节，“检查服务器线程（进程）信息”。此列的值对应于客户端/服务器协议的`COM_*`xxx`*`命令和`Com_*`xxx`*`状态变量。请参见第 7.1.10 节，“服务器状态变量”。

+   `时间`

    线程处于当前状态的秒数。对于副本 SQL 线程，该值是最后一个复制事件的时间戳与副本主机的实时时间之间的秒数。请参见第 19.2.3 节，“复制线程”。

+   `状态`

    表示线程正在执行的操作、事件或状态。有关`State`值的描述，请参见第 10.14 节，“检查服务器线程（进程）信息”。

    大多数状态对应非常快速的操作。如果一个线程停留在某个状态很多秒钟，可能存在需要调查的问题。

+   `信息`

    线程正在执行的语句，如果没有执行语句则为`NULL`。该语句可能是发送给服务器的语句，或者如果该语句执行其他语句，则为最内层语句。例如，如果一个`CALL`语句执行一个正在执行`SELECT`语句的存储过程，那么`信息`值显示`SELECT`语句。
