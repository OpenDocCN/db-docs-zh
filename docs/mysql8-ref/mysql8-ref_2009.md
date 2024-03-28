# 29.10 性能模式语句摘要和采样

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-digests.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-digests.html)

MySQL 服务器能够维护语句摘要信息。摘要过程将每个 SQL 语句转换为规范化形式（语句摘要），并从规范化结果计算一个 SHA-256 哈希值（摘要哈希值）。规范化允许类似的语句被分组和总结，以公开关于服务器正在执行的语句类型及其频率的信息。对于每个摘要，存储生成该摘要的代表性语句作为样本。本节描述了语句摘要和采样的发生方式以及它们的用途。

无论性能模式是否可用，解析器都会进行摘要处理，以便其他功能如 MySQL 企业防火墙和查询重写插件可以访问语句摘要。

+   语句摘要概念

+   性能模式中的语句摘要

+   语句摘要内存使用

+   语句采样

### 语句摘要概念

当解析器接收到一个 SQL 语句时，如果需要该摘要，则会计算一个语句摘要，如果以下任一条件为真，则为真：

+   启用了性能模式摘要工具

+   启用了 MySQL 企业防火墙

+   启用了查询重写插件

解析器还被 `STATEMENT_DIGEST_TEXT()` 和 `STATEMENT_DIGEST()` 函数使用，应用程序可以调用这些函数从 SQL 语句中计算规范化语句摘要和摘要哈希值。

`max_digest_length` 系统变量的值确定了每个会话用于计算规范化语句摘要的最大字节数。一旦在摘要计算过程中使用了这么多空间，就会发生截断：从解析语句中收集的标记不再被收集或计入其摘要值。只有在解析标记之后的那么多字节之后才有所不同的语句会产生相同的规范化语句摘要，并且如果进行比较或用于摘要统计，则被视为相同。

警告

将`max_digest_length`系统变量设置为零会禁用摘要生成，这也会禁用需要摘要的服务器功能。

计算规范化语句后，从中计算 SHA-256 哈希值。此外：

+   如果启用了 MySQL 企业防火墙，则会调用它，并且计算的摘要对其可用。

+   如果启用了任何查询重写插件，则会调用它，并且语句摘要和摘要值对其可用。

+   如果性能模式启用了摘要仪器，它会复制规范化的语句摘要，为其分配最多`performance_schema_max_digest_length`字节。因此，如果`performance_schema_max_digest_length`小于`max_digest_length`，则复制相对于原始文本被截断。规范化的语句摘要的副本存储在适当的性能模式表中，以及从原始规范化语句计算的 SHA-256 哈希值。 （如果性能模式相对于原始文本截断其规范化语句摘要的副本，则不会重新计算 SHA-256 哈希值。）

语句规范化将语句文本转换为更标准化的摘要字符串表示形式，保留一般语句结构同时删除不影响结构的信息：

+   保留对象标识符，如数据库和表名。

+   文本值被转换为参数标记。规范化的语句不保留名称、密码、日期等信息。

+   注释被移除，空格被调整。

考虑这些语句：

```sql
SELECT * FROM orders WHERE customer_id=10 AND quantity>20
SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
```

为了规范化这些语句，解析器将数据值替换为`?`并调整空格。两个语句产生相同的规范化形式，因此被认为是“相同的”：

```sql
SELECT * FROM orders WHERE customer_id = ? AND quantity > ?
```

规范化的语句包含较少信息，但仍代表原始语句。其他类似的具有不同数据值的语句具有相同的规范化形式。

现在考虑这些语句：

```sql
SELECT * FROM customers WHERE customer_id = 1000
SELECT * FROM orders WHERE customer_id = 1000
```

在这种情况下，规范化的语句不同，因为对象标识符不同：

```sql
SELECT * FROM customers WHERE customer_id = ?
SELECT * FROM orders WHERE customer_id = ?
```

如果规范化产生的语句超过摘要缓冲区中可用的空间（由`max_digest_length`确定），则会发生截断，并且文本以“...”结尾。只有在“...”后面部分不同的长规范化语句被视为相同。考虑这些语句：

```sql
SELECT * FROM mytable WHERE cola = 10 AND colb = 20
SELECT * FROM mytable WHERE cola = 10 AND colc = 20
```

如果截断恰好发生在`AND`之后，则这两个语句具有相同的规范化形式：

```sql
SELECT * FROM mytable WHERE cola = ? AND ...
```

在这种情况下，第二列名称的差异被忽略，两个语句被视为相同。

### 性能模式中的语句摘要

在性能模式中，语句摘要涉及以下元素：

+   `setup_consumers` 表中的 `statements_digest` 消费者控制性能模式是否维护摘要信息。参见 语句摘要消费者。

+   语句事件表（`events_statements_current`，`events_statements_history` 和 `events_statements_history_long`）具有用于存储规范语句摘要和相应摘要 SHA-256 哈希值的列：

    +   `DIGEST_TEXT` 是规范语句摘要的文本。这是原始规范语句的副本，计算到最大 `max_digest_length` 字节，根据需要进一步截断为 `performance_schema_max_digest_length` 字节。

    +   `DIGEST` 是从原始规范语句计算得出的摘要 SHA-256 哈希值。

    参见 第 29.12.6 节，“性能模式语句事件表”。

+   `events_statements_summary_by_digest` 摘要表提供了聚合的语句摘要信息。该表按 `SCHEMA_NAME` 和 `DIGEST` 组合对语句信息进行聚合。性能模式使用 SHA-256 哈希值进行聚合，因为它们计算速度快，并且具有有利的统计分布，可以最小化碰撞。参见 第 29.12.20.3 节，“语句摘要表”。

一些性能表具有一个列，用于存储计算摘要的原始 SQL 语句：

+   `events_statements_current`，`events_statements_history` 和 `events_statements_history_long` 语句事件表的 `SQL_TEXT` 列。

+   `events_statements_summary_by_digest` 摘要表的 `QUERY_SAMPLE_TEXT` 列。

默认情况下，语句显示的最大空间为 1024 字节。要更改此值，请在服务器启动时设置 `performance_schema_max_sql_text_length` 系统变量。更改会影响所有上述列所需的存储空间。

`performance_schema_max_digest_length` 系统变量确定性能模式中用于摘要值存储的每个语句的最大字节数。然而，由于语句元素（如关键字和文字值）的内部编码，语句摘要的显示长度可能长于可用缓冲区大小。因此，从语句事件表的 `DIGEST_TEXT` 列中选择的值可能会超过 `performance_schema_max_digest_length` 的值。

`events_statements_summary_by_digest` 摘要表提供了服务器执行的语句的概要。它显示了应用程序执行的语句类型和频率。应用程序开发人员可以将此信息与表中的其他信息一起使用，评估应用程序的性能特征。例如，显示等待时间、锁定时间或索引使用的表列可能突出显示效率低下的查询类型。这使开发人员了解应用程序哪些部分需要关注。

`events_statements_summary_by_digest` 摘要表具有固定大小。默认情况下，性能模式在启动时估计要使用的大小。要明确指定表大小，请在服务器启动时设置 `performance_schema_digests_size` 系统变量。如果表变满，性能模式将具有 `SCHEMA_NAME` 和 `DIGEST` 值不匹配现有表中值的语句分组到一个特殊行中，该行的 `SCHEMA_NAME` 和 `DIGEST` 设置为 `NULL`。这允许计算所有语句。但是，如果特殊行占执行的语句的显著百分比，可能需要通过增加 `performance_schema_digests_size` 来增加摘要表大小。

### 语句摘要内存使用

对于生成仅在结尾处有所不同的非常长语句的应用程序，增加`max_digest_length`可以计算出区分否则会聚合到相同摘要的语句的摘要。相反，减少`max_digest_length`会导致服务器将更少的内存用于摘要存储，但增加较长语句聚合到相同摘要的可能性。管理员应牢记，较大的值会导致相应增加的内存需求，特别是对于涉及大量同时会话的工作负载（服务器为每个会话分配`max_digest_length`字节）。

如前所述，由解析器计算的规范化语句摘要受限于最多`max_digest_length`字节，而性能模式中存储的规范化语句摘要使用`performance_schema_max_digest_length`字节。关于`max_digest_length`和`performance_schema_max_digest_length`的相对值，以下内存使用考虑适用：

+   如果`max_digest_length`小于`performance_schema_max_digest_length`：

    +   除了性能模式之外的服务器功能使用占用最多`max_digest_length`字节的规范化语句摘要。

    +   性能模式不会进一步截断存储的规范化语句摘要，但为每个摘要分配比`max_digest_length`字节更多的内存，这是不必要的。

+   如果`max_digest_length`等于`performance_schema_max_digest_length`：

    +   除了性能模式之外的服务器功能使用占用最多`max_digest_length`字节的规范化语句摘要。

    +   性能模式不会进一步截断存储的规范化语句摘要，并为每个摘要分配与`max_digest_length`字节相同的内存。

+   如果`max_digest_length`大于`performance_schema_max_digest_length`：

    +   Performance Schema 之外的服务器功能使用占用`max_digest_length`字节的规范语句摘要。

    +   Performance Schema 进一步截断其存储的规范语句摘要，并为每个摘要分配少于`max_digest_length`字节的内存。

因为 Performance Schema 语句事件表可能存储许多摘要，所以将`performance_schema_max_digest_length`设置为小于`max_digest_length`使管理员能够平衡这些因素：

+   在 Performance Schema 之外的服务器功能需要长规范语句摘要

+   许多并发会话，每个会话分配摘要计算内存

+   在存储许多语句摘要时，需要限制 Performance Schema 语句事件表的内存消耗

`performance_schema_max_digest_length`设置不是每个会话，而是每个语句，一个会话可以在`events_statements_history`表中存储多个语句。在此表中的典型语句数量为每个会话 10 个，因此每个会话仅针对此表消耗了`performance_schema_max_digest_length`值所示的内存的 10 倍。

此外，全局收集了许多语句（和摘要），最显著的是在`events_statements_history_long`表中。在这里，存储的*N*语句消耗了*N*倍于`performance_schema_max_digest_length`值所示内存的内存。

要评估用于 SQL 语句存储和摘要计算的内存量，请使用`SHOW ENGINE PERFORMANCE_SCHEMA STATUS`语句，或监视这些指标：

```sql
mysql> SELECT NAME
       FROM performance_schema.setup_instruments
       WHERE NAME LIKE '%.sqltext';
+------------------------------------------------------------------+
| NAME                                                             |
+------------------------------------------------------------------+
| memory/performance_schema/events_statements_history.sqltext      |
| memory/performance_schema/events_statements_current.sqltext      |
| memory/performance_schema/events_statements_history_long.sqltext |
+------------------------------------------------------------------+

mysql> SELECT NAME
       FROM performance_schema.setup_instruments
       WHERE NAME LIKE 'memory/performance_schema/%.tokens';
+----------------------------------------------------------------------+
| NAME                                                                 |
+----------------------------------------------------------------------+
| memory/performance_schema/events_statements_history.tokens           |
| memory/performance_schema/events_statements_current.tokens           |
| memory/performance_schema/events_statements_summary_by_digest.tokens |
| memory/performance_schema/events_statements_history_long.tokens      |
+----------------------------------------------------------------------+
```

### 语句抽样

性能模式使用语句取样来收集在`events_statements_summary_by_digest`表中生成每个摘要值的代表性语句。这些列存储样本语句信息：`QUERY_SAMPLE_TEXT`（语句的文本），`QUERY_SAMPLE_SEEN`（语句被看到的时间）和`QUERY_SAMPLE_TIMER_WAIT`（语句等待或执行时间）。每次选择样本语句时，性能模式都会更新这三列。

当插入新表行时，生成行摘要值的语句将作为与摘要相关联的当前样本语句存储。此后，当服务器看到具有相同摘要值的其他语句时，它将确定是否使用新语句替换当前样本语句（即是否重新取样）。重新取样策略基于当前样本语句和新语句的比较等待时间，以及可选的当前样本语句的年龄：

+   基于等待时间的重新取样：如果新语句的等待时间大于当前样本语句的等待时间，则新语句将成为当前样本语句。

+   基于年龄的重新取样：如果`performance_schema_max_digest_sample_age`系统变量的值大于零，并且当前样本语句的年龄超过该秒数，则当前语句被视为“太旧”，新语句将替换它。即使新语句的等待时间小于当前样本语句的等待时间，也会发生这种情况。

默认情况下，`performance_schema_max_digest_sample_age`为 60 秒（1 分钟）。要更改样本语句由于年龄而“过期”的速度，增加或减少该值。要禁用基于年龄的重新取样策略的部分，请将`performance_schema_max_digest_sample_age`设置为 0。
