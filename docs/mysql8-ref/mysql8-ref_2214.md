> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-truncate-all-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-truncate-all-tables.html)

#### 30.4.4.24 `ps_truncate_all_tables()` 过程

截断所有性能模式摘要表，重置所有聚合仪表作为快照的指令。生成一个结果集，指示截断了多少个表。

##### 参数

+   `in_verbose BOOLEAN`：是否在执行之前显示每个`TRUNCATE TABLE`语句。

##### 示例

```sql
mysql> CALL sys.ps_truncate_all_tables(FALSE);
+---------------------+
| summary             |
+---------------------+
| Truncated 49 tables |
+---------------------+
```
