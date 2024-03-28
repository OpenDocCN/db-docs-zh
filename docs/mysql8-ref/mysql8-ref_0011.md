# 1.6 MySQL 标准兼容性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/compatibility.html`](https://dev.mysql.com/doc/refman/8.0/en/compatibility.html)

1.6.1 MySQL 对标准 SQL 的扩展

1.6.2 MySQL 与标准 SQL 的差异

1.6.3 MySQL 如何处理约束

本节描述了 MySQL 与 ANSI/ISO SQL 标准的关系。MySQL Server 对 SQL 标准有许多扩展，在这里您可以找到它们是什么以及如何使���它们。您还可以找到有关 MySQL Server 缺失功能的信息，以及如何解决其中一些差异。

SQL 标准自 1986 年以来一直在不断发展，存在多个版本。在本手册中，“SQL-92” 指的是 1992 年发布的标准。“SQL:1999”、“SQL:2003”、“SQL:2008” 和 “SQL:2011” 分别指的是相应年份发布的标准版本，最后一个是最新版本。我们使用短语 “SQL 标准” 或 “标准 SQL” 来表示任何时候 SQL 标准的当前版本。

我们的产品的主要目标之一是继续努力遵守 SQL 标准，但不会牺牲速度或可靠性。如果这大大提高了 MySQL Server 对我们大部分用户群体的可用性，我们不会害怕添加 SQL 扩展或非 SQL 功能的支持。`HANDLER` 接口就是这种策略的一个例子。请参阅 第 15.2.5 节，“HANDLER 语句”。

我们继续支持事务性和非事务性数据库，以满足关键任务的 24/7 使用和大量 Web 或日志使用。

MySQL Server 最初设计用于在小型计算机系统上处理中等大小的数据库（10-100 百万行，或每个表约 100MB）。如今，MySQL Server 处理 TB 级别大小的数据库。

我们不针对实时支持，尽管 MySQL 复制功能提供了重要的功能。

MySQL 支持 ODBC 0 到 3.51 级别。

MySQL 支持使用 `NDBCLUSTER` 存储引擎进行高可用性数据库集群。请参阅 第二十五章，“MySQL NDB Cluster 8.0”。

我们实现了支持大部分 W3C XPath 标准的 XML 功能。请参阅 第 14.11 节，“XML 函数”。

MySQL 支持由 RFC 7159 定义的本机 JSON 数据类型，并基于 ECMAScript 标准（ECMA-262）。请参阅 第 13.5 节，“JSON 数据类型”。MySQL 还实现了 SQL:2016 标准的一个预发布草案中指定的 SQL/JSON 函数子集；更多信息请参阅 第 14.17 节，“JSON 函数”。

### 选择 SQL 模式

MySQL 服务器可以在不同的 SQL 模式下运行，并且可以根据 `sql_mode` 系统变量的值为不同的客户端应用这些模式。数据库管理员可以将全局 SQL 模式设置为符合站点服务器操作要求的模式，每个应用程序可以将其会话 SQL 模式设置为其自身的要求。

模式影响 MySQL 支持的 SQL 语法和执行的数据验证检查。这使得在不同环境中使用 MySQL 以及与其他数据库服务器一起使用 MySQL 更容易。

有关设置 SQL 模式的更多信息，请参阅 Section 7.1.11, “Server SQL Modes”。

### 在 ANSI 模式下运行 MySQL

要在 ANSI 模式下运行 MySQL 服务器，请使用 `--ansi` 选项启动 **mysqld**。在 ANSI 模式下运行服务器与以下选项启动它相同：

```sql
--transaction-isolation=SERIALIZABLE --sql-mode=ANSI
```

要在运行时实现相同的效果，请执行以下两个语句：

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET GLOBAL sql_mode = 'ANSI';
```

您可以看到将 `sql_mode` 系统变量设置为 `'ANSI'` 会启用所有与 ANSI 模式相关的 SQL 模式选项，如下所示：

```sql
mysql> SET GLOBAL sql_mode='ANSI';
mysql> SELECT @@GLOBAL.sql_mode;
        -> 'REAL_AS_FLOAT,PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,ANSI'
```

使用 `--ansi` 在 ANSI 模式下运行服务器与将 SQL 模式设置为 `'ANSI'` 不完全相同，因为 `--ansi` 选项还设置了事务隔离级别。

请参阅 Section 7.1.7, “Server Command Options”。
