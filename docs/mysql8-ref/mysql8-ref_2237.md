> [`dev.mysql.com/doc/refman/8.0/en/sys-version-major.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-version-major.html)

#### 30.4.5.20 `version_major()`函数

此函数返回 MySQL 服务器的主要版本。

##### 参数

无。

##### 返回值

一个`TINYINT UNSIGNED`值。

##### 示例

```sql
mysql> SELECT VERSION(), sys.version_major();
+--------------+---------------------+
| VERSION()    | sys.version_major() |
+--------------+---------------------+
| 8.0.26-debug |                   8 |
+--------------+---------------------+
```
