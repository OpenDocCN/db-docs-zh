> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-version-minor.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-version-minor.html)

#### 30.4.5.21 版本次要() 函数

此函数返回 MySQL 服务器的次要版本。

##### 参数

无。

##### 返回值

一个 `TINYINT UNSIGNED` 值。

##### 示例

```sql
mysql> SELECT VERSION(), sys.version_minor();
+--------------+---------------------+
| VERSION()    | sys.version_minor() |
+--------------+---------------------+
| 8.0.26-debug |                   0 |
+--------------+---------------------+
```
