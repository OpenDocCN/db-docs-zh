# 10.14.1 访问进程列表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/processlist-access.html`](https://dev.mysql.com/doc/refman/8.0/en/processlist-access.html)

以下讨论列举了进程信息的来源，查看进程信息所需的权限，并描述了进程列表条目的内容。

+   进程信息来源

+   访问进程列表所需的权限

+   进程列表条目内容

#### 进程信息来源

进程信息可从以下来源获取：

+   `SHOW PROCESSLIST` 语句：Section 15.7.7.29, “SHOW PROCESSLIST 语句”

+   **mysqladmin processlist** 命令：Section 6.5.2, “mysqladmin — MySQL 服务器管理程序”

+   `INFORMATION_SCHEMA` `PROCESSLIST` 表：Section 28.3.23, “INFORMATION_SCHEMA PROCESSLIST 表”

+   Performance Schema `processlist` 表：Section 29.12.21.7, “processlist 表”

+   Performance Schema `threads` 表列名以 `PROCESSLIST_` 为前缀：Section 29.12.21.8, “threads 表”

+   `sys` schema `processlist` 和 `session` 视图：Section 30.4.3.22, “processlist 和 x$processlist 视图”, 和 Section 30.4.3.33, “session 和 x$session 视图”

`threads` 表与 `SHOW PROCESSLIST`, `INFORMATION_SCHEMA` `PROCESSLIST`, 和 **mysqladmin processlist** 相比如下：

+   访问`threads`表不需要互斥锁，并且对服务器性能影响很小。其他来源会产生负面性能影响，因为它们需要互斥锁。

    注意

    截至 MySQL 8.0.22 版本，基于性能模式`processlist`表的另一种`SHOW PROCESSLIST`实现可用，类似于`threads`表，不需要互斥锁，并具有更好的性能特性。详情请参见第 29.12.21.7 节，“The processlist Table”。

+   `threads`表显示后台线程，其他来源不显示。它还为每个线程提供其他来源不提供的附加信息，例如线程是前台线程还是后台线程，以及与线程关联的服务器内部位置。这意味着`threads`表可用于监控其他来源无法监控的线程活动。

+   您可以启用或禁用性能模式线程监控，如第 29.12.21.8 节，“The threads Table”所述。

出于这些原因，使用其他线程信息源之一进行服务器监控的数据库管理员可能希望改为使用`threads`表进行监控。

`sys`模式`processlist`视图以更易访问的格式呈现性能模式`threads`表中的信息。`sys`模式`session`视图呈现有关用户会话的信息，类似于`sys`模式`processlist`视图，但过滤掉后台进程。

#### 访问进程列表所需的权限

对于大多数进程信息来源，如果具有`PROCESS`权限，则可以查看所有线程，即使属于其他用户。否则（没有`PROCESS`权限），非匿名用户可以访问有关自己线程的信息，但不能访问其他用户的线程，匿名用户无法访问线程信息。

性能模式`threads`表还提供线程信息，但表访问使用不同的权限模型。请参阅第 29.12.21.8 节，“threads 表”。

#### 进程列表条目内容

每个进程列表条目包含几个信息片段。以下列表使用`SHOW PROCESSLIST`输出中的标签描述它们。其他进程信息源使用类似的标签。

+   `Id`是与线程关联的客户端的连接标识符。

+   `User`和`Host`表示与线程关联的帐户。

+   `db`是线程的默认数据库，如果没有选择任何数据库，则为`NULL`。

+   `Command`和`State`表示线程正在执行的操作。

    大多数状态对应非常快速的操作。如果线程在给定状态下停留了很多秒钟，可能存在需要调查的问题。

    以下各节列出了可能的`Command`值，以及按类别分组的`State`值。对于其中一些值，其含义是不言自明的。对于其他值，提供了额外的描述。

    注意

    检查进程列表信息的应用程序应该注意，命令和状态可能会发生变化。

+   `Time`表示线程在当前状态下已经存在的时间。在某些情况下，线程的当前时间概念可能会发生变化：线程可以使用`SET TIMESTAMP = *`value`*`更改时间。对于复制 SQL 线程，该值是最后一个复制事件的时间戳与复制主机的实际时间之间的秒数。请参阅第 19.2.3 节，“复制线程”。

+   `Info`表示线程正在执行的语句，如果没有执行任何语句，则为`NULL`。对于`SHOW PROCESSLIST`，此值仅包含语句的前 100 个字符。要查看完整的语句，请使用`SHOW FULL PROCESSLIST`（或查询不同的进程信息源）。
