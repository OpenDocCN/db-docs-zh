> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-rules-db-options.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-rules-db-options.html)

#### 19.2.5.1 数据库级复制和二进制日志选项的评估

在评估复制选项时，复制开始时首先检查是否有适用的[`--replicate-do-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-do-db)或[`--replicate-ignore-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-ignore-db)选项。当使用[`--binlog-do-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_binlog-do-db)或[`--binlog-ignore-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db)时，该过程类似，但选项在源上进行检查。

被检查是否匹配的数据库取决于正在处理的语句的二进制日志格式。如果使用行格式记录了该语句，则被更改数据的数据库是被检查的数据库。如果使用语句格式记录了该语句，则默认数据库（使用[`USE`](https://dev.mysql.com/doc/refman/8.0/en/use.html "15.8.4 USE Statement")语句指定）是被检查的数据库。

注意

只有使用行格式的 DML 语句才能被记录。DDL 语句始终作为语句记录，即使[`binlog_format=ROW`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_format)。因此，所有 DDL 语句都根据基于语句的复制规则进行过滤。这意味着你必须使用[`USE`](https://dev.mysql.com/doc/refman/8.0/en/use.html "15.8.4 USE Statement")语句显式选择默认数据库，以便应用 DDL 语句。

对于复制，所涉及的步骤如下：

1.  使用了哪种日志格式？

    +   **语句。** 测试默认数据库。

    +   **行。** 测试受更改影响的数据库。

1.  是否有任何[`--replicate-do-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-do-db)选项？

    +   **是。** 数据库是否与它们中的任何一个匹配？

        +   **是。** 继续到第 4 步。

        +   **否。** 忽略更新并退出。

    +   **否。** 继续到第 3 步。

1.  是否有任何[`--replicate-ignore-db`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-ignore-db)选项？

    +   **是。** 数据库是否与它们中的任何一个匹配？

        +   **是。** 忽略更新并退出。

        +   **否。** 继续到第 4 步。

    +   **否。** 继续到第 4 步。

1.  如果有的话，继续检查表级复制选项。有关如何检查这些选项的描述，请参见[Section 19.2.5.2, “Evaluation of Table-Level Replication Options”](https://dev.mysql.com/doc/refman/8.0/en/replication-rules-table-options.html "19.2.5.2 Evaluation of Table-Level Replication Options")。

    重要

    在这个阶段仍然允许的语句实际上尚未执行。直到所有表级选项（如果有）也被检查，并且该过程的结果允许执行该语句为止，该语句才会被执行。

对于二进制日志记录，所涉及的步骤如下：

1.  是否有`--binlog-do-db`或`--binlog-ignore-db`选项？

    +   **是的。** 继续到第 2 步。

    +   **否。** 记录该语句并退出。

1.  是否存在默认数据库（是否已通过`USE`选择了任何数据库）？

    +   **是的。** 继续到第 3 步。

    +   **否。** 忽略该语句并退出。

1.  存在默认数据库。是否有`--binlog-do-db`选项？

    +   **是的。** 有任何匹配的数据库吗？

        +   **是的。** 记录该语句并退出。

        +   **否。** 忽略该语句并退出。

    +   **否。** 继续到第 4 步。

1.  任何`--binlog-ignore-db`选项是否与数据库匹配？

    +   **是的。** 忽略该语句并退出。

    +   **否。** 记录该语句并退出。

重要提示

对于基于语句的日志记录，在刚才给出的规则中对`CREATE DATABASE`、`ALTER DATABASE`和`DROP DATABASE`语句做了例外。在这些情况下，正在*创建、更改或删除*的数据库将替换默认数据库，以确定是否记录或忽略更新。

`--binlog-do-db`有时可能意味着“忽略其他数据库”。例如，当使用基于语句的日志记录时，仅使用`--binlog-do-db=sales`的服务器不会将与默认数据库不同的`sales`的语句写入二进制日志。当使用相同选项的基于行的日志记录时，服务器仅记录更改`sales`中数据的更新。
