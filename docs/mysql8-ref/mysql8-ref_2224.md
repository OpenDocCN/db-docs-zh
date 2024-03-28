> [`dev.mysql.com/doc/refman/8.0/en/sys-list-add.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-list-add.html)

#### 30.4.5.7 `list_add()`函数

将一个值添加到逗号分隔的值列表中并返回结果。

这个函数和`list_drop()` Function")对于操作系统变量的值非常有用，比如`sql_mode`和`optimizer_switch`，它们接受一个逗号分隔的值列表。

##### 参数

+   `in_list TEXT`：需要修改的列表。

+   `in_add_value TEXT`：要添加到列表中的值。

##### 返回值

一个`TEXT`值。

##### 示例

```sql
mysql> SELECT @@sql_mode;
+----------------------------------------+
| @@sql_mode                             |
+----------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES |
+----------------------------------------+
mysql> SET @@sql_mode = sys.list_add(@@sql_mode, 'NO_ENGINE_SUBSTITUTION');
mysql> SELECT @@sql_mode;
+---------------------------------------------------------------+
| @@sql_mode                                                    |
+---------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |
+---------------------------------------------------------------+
mysql> SET @@sql_mode = sys.list_drop(@@sql_mode, 'ONLY_FULL_GROUP_BY');
mysql> SELECT @@sql_mode;
+--------------------------------------------+
| @@sql_mode                                 |
+--------------------------------------------+
| STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |
+--------------------------------------------+
```
