# 18.4 CSV 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/csv-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/csv-storage-engine.html)

18.4.1 修复和检查 CSV 表

18.4.2 CSV 限制

`CSV`存储引擎使用逗号分隔值格式将数据存储在文本文件中。

`CSV`存储引擎始终编译到 MySQL 服务器中。

要查看`CSV`引擎的源代码，请查看 MySQL 源代码分发中的`storage/csv`目录。

当你创建一个`CSV`表时，服务器会创建一个以表名开头并以`.CSV`扩展名结尾的纯文本数据文件。当你将数据存储到表中时，存储引擎会将其保存为逗号分隔值格式的数据文件。

```sql
mysql> CREATE TABLE test (i INT NOT NULL, c CHAR(10) NOT NULL)
       ENGINE = CSV;
Query OK, 0 rows affected (0.06 sec)

mysql> INSERT INTO test VALUES(1,'record one'),(2,'record two');
Query OK, 2 rows affected (0.05 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM test;
+---+------------+
| i | c          |
+---+------------+
| 1 | record one |
| 2 | record two |
+---+------------+
2 rows in set (0.00 sec)
```

创建一个`CSV`表还会创建一个相应的元文件，用于存储表的状态和表中存在的行数。该文件的名称与表名相同，扩展名为`CSM`。

如果你检查执行上述语句创建的数据库目录中的`test.CSV`文件，其内容应该如下所示：

```sql
"1","record one"
"2","record two"
```

这种格式可以被电子表格应用程序（如 Microsoft Excel）读取，甚至写入。
