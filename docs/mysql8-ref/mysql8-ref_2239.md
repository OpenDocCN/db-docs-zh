> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-version-patch.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-version-patch.html)

#### 30.4.5.22 版本 _patch() 函数

此函数返回 MySQL 服务器的补丁发布版本。

##### 参数

无。

##### 返回值

一个`TINYINT UNSIGNED`值。

##### 示例

```sql
mysql> SELECT VERSION(), sys.version_patch();
+--------------+---------------------+
| VERSION()    | sys.version_patch() |
+--------------+---------------------+
| 8.0.26-debug |                  26 |
+--------------+---------------------+
```
