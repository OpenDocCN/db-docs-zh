# 10.8.1 使用 EXPLAIN 优化查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-explain.html`](https://dev.mysql.com/doc/refman/8.0/en/using-explain.html)

`EXPLAIN`语句提供有关 MySQL 如何执行语句的信息：

+   `EXPLAIN`适用于`SELECT`、`DELETE`、`INSERT`、`REPLACE`和`UPDATE`语句。

+   当使用`EXPLAIN`解释可解释的语句时，MySQL 会显示有关语句执行计划的优化器信息。也就是说，MySQL 会解释它将如何处理该语句，包括有关表如何连接以及顺序的信息。有关使用`EXPLAIN`获取执行计划信息的信息，请参见 Section 10.8.2, “EXPLAIN Output Format”。

+   当使用`EXPLAIN`与`FOR CONNECTION *connection_id*`而不是可解释的语句一起使用时，它会显示在指定连接中执行的语句的执行计划。请参见 Section 10.8.4, “Obtaining Execution Plan Information for a Named Connection”。

+   对于`SELECT`语句，`EXPLAIN`生成可使用`SHOW WARNINGS`显示的附加执行计划信息。请参见 Section 10.8.3, “Extended EXPLAIN Output Format”。

+   `EXPLAIN`对于检查涉及分区表的查询很有用。请参见 Section 26.3.5, “Obtaining Information About Partitions”。

+   `FORMAT`选项可用于选择输出格式。`TRADITIONAL`以表格格式呈现输出。如果没有`FORMAT`选项，则默认为此格式。`JSON`格式以 JSON 格式显示信息。

借助`EXPLAIN`，您可以看到应该向表中添加索引以使语句通过使用索引查找行而更快执行的位置。您还可以使用`EXPLAIN`来检查优化器是否以最佳顺序连接表。为了向优化器提供提示，使用与`SELECT`语句中表的命名顺序相对应的连接顺序，可以在语句开头使用`SELECT STRAIGHT_JOIN`而不仅仅是`SELECT`。（参见 Section 15.2.13, “SELECT Statement”。）然而，`STRAIGHT_JOIN`可能会阻止索引的使用，因为它禁用了半连接转换。请参见 Section 10.2.2.1, “Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations”。

优化器跟踪有时可能提供与`EXPLAIN`互补的信息。然而，优化器跟踪格式和内容可能会在版本之间发生变化。有关详细信息，请参见 MySQL Internals: Tracing the Optimizer。

如果您发现索引没有被使用，而您认为它们应该被使用，请运行`ANALYZE TABLE`来更新表统计信息，例如键的基数，这可能会影响优化器的选择。请参见 Section 15.7.3.1, “ANALYZE TABLE Statement”。

注意

`EXPLAIN`也可以用于获取表中列的信息。`EXPLAIN *`tbl_name`*`与`DESCRIBE *`tbl_name`*`和`SHOW COLUMNS FROM *`tbl_name`*`是同义的。有关更多信息，请参见 Section 15.8.1, “DESCRIBE Statement”和 Section 15.7.7.5, “SHOW COLUMNS Statement”。
