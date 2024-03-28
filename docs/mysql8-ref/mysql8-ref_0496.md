> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-copying-database.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-copying-database.html)

#### 9.4.5.1 复制数据库

```sql
$> mysqldump db1 > dump.sql
$> mysqladmin create db2
$> mysql db2 < dump.sql
```

在 **mysqldump** 命令行中不要使用 `--databases`，因为这会导致在转储文件中包含 `USE db1`，这会覆盖在 **mysql** 命令行上命名 `db2` 的效果。
