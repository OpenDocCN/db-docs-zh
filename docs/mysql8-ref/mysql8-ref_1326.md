# 18.9 `EXAMPLE`存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/example-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/example-storage-engine.html)

`EXAMPLE`存储引擎是一个什么都不做的存根引擎。它的目的是作为 MySQL 源代码中的一个示例，展示如何开始编写新的存储引擎。因此，它主要是为开发人员感兴趣。

如果您从源代码构建 MySQL，并希望启用`EXAMPLE`存储引擎，请使用**CMake**调用`-DWITH_EXAMPLE_STORAGE_ENGINE`选项。

要查看`EXAMPLE`引擎的源代码，请查看 MySQL 源代码分发中的`storage/example`目录。

当您创建一个`EXAMPLE`表时，不会创建任何文件。表中无法存储任何数据。检索将返回一个空结果。

```sql
mysql> CREATE TABLE test (i INT) ENGINE = EXAMPLE;
Query OK, 0 rows affected (0.78 sec)

mysql> INSERT INTO test VALUES(1),(2),(3);
ERROR 1031 (HY000): Table storage engine for 'test' doesn't »
                    have this option 
mysql> SELECT * FROM test;
Empty set (0.31 sec)
```

`EXAMPLE`存储引擎不支持索引。

`EXAMPLE`存储引擎不支持分区。
