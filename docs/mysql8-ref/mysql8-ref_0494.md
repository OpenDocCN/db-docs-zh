# 9.4.4 重新加载分隔文本格式备份

> 原文：[`dev.mysql.com/doc/refman/8.0/en/reloading-delimited-text-dumps.html`](https://dev.mysql.com/doc/refman/8.0/en/reloading-delimited-text-dumps.html)

对于使用**mysqldump --tab**生成的备份，每个表在输出目录中由一个包含表的`CREATE TABLE`语句的`.sql`文件和一个包含表数据的`.txt`文件表示。要重新加载表，首先进入输出目录。然后使用**mysql**处理`.sql`文件以创建空表，并处理`.txt`文件以将数据加载到表中：

```sql
$> mysql db1 < t1.sql
$> mysqlimport db1 t1.txt
```

除了使用**mysqlimport**加载数据文件之外，还可以在**mysql**客户端内部使用`LOAD DATA`语句：

```sql
mysql> USE db1;
mysql> LOAD DATA INFILE 't1.txt' INTO TABLE t1;
```

如果在最初转储表时使用了任何数据格式化选项**mysqldump**，则必须在使用**mysqlimport**或`LOAD DATA`时使用相同的选项，以确保正确解释数据文件内容：

```sql
$> mysqlimport --fields-terminated-by=,
         --fields-enclosed-by='"' --lines-terminated-by=0x0d0a db1 t1.txt
```

或：

```sql
mysql> USE db1;
mysql> LOAD DATA INFILE 't1.txt' INTO TABLE t1
       FIELDS TERMINATED BY ',' FIELDS ENCLOSED BY '"'
       LINES TERMINATED BY '\r\n';
```
