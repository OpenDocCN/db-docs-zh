> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-ndb-sync-pending-objects-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-ndb-sync-pending-objects-table.html)

#### 29.12.12.1 ndb_sync_pending_objects 表

这个表提供了关于已检测到不匹配并正在等待在`NDB`字典和 MySQL 数据字典之间同步的数据库对象的信息。

有关等待同步的`NDB`数据库对象的示例信息：

```sql
mysql> SELECT * FROM performance_schema.ndb_sync_pending_objects;
+-------------+------+----------------+
| SCHEMA_NAME | NAME |  TYPE          |
+-------------+------+----------------+
| NULL        | lg1  |  LOGFILE GROUP |
| NULL        | ts1  |  TABLESPACE    |
| db1         | NULL |  SCHEMA        |
| test        | t1   |  TABLE         |
| test        | t2   |  TABLE         |
| test        | t3   |  TABLE         |
+-------------+------+----------------+
```

`ndb_sync_pending_objects`表具有以下列：

+   `SCHEMA_NAME`: 等待同步的对象所在模式（数据库）的名称；对于表空间和日志文件组，此处为`NULL`。

+   `NAME`: 等待同步的对象的名称；如果对象是模式，则为`NULL`。

+   `TYPE`: 等待同步的对象的类型；可以是`LOGFILE GROUP`、`TABLESPACE`、`SCHEMA`或`TABLE`之一。

`ndb_sync_pending_objects`表是在 NDB 8.0.21 中添加的。
