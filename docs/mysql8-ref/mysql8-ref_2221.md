> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-format-path.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-format-path.html)

#### 30.4.5.4 `format_path()`函数

给定一个路径名，返回在按顺序替换以下系统变量的值匹配的子路径后的修改路径名：

```sql
datadir
tmpdir
slave_load_tmpdir or replica_load_tmpdir
innodb_data_home_dir
innodb_log_group_home_dir
innodb_undo_directory
basedir
```

与系统变量*`sysvar`*的值匹配的值将被替换为字符串`@@GLOBAL.*`sysvar`*`。

##### 参数

+   `path VARCHAR(512)`: 要格式化的路径名。

##### 返回数值

一个`VARCHAR(512) CHARACTER SET utf8mb3`的数值。

##### 示例

```sql
mysql> SELECT sys.format_path('/usr/local/mysql/data/world/City.ibd');
+---------------------------------------------------------+
| sys.format_path('/usr/local/mysql/data/world/City.ibd') |
+---------------------------------------------------------+
| @@datadir/world/City.ibd                                |
+---------------------------------------------------------+
```
