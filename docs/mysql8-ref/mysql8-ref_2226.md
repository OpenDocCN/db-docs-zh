> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-ps-is-account-enabled.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-is-account-enabled.html)

#### 30.4.5.9 `ps_is_account_enabled()` 函数

返回 `YES` 或 `NO` 表示给定账户的性能模式仪器是否已启用。

##### 参数

+   `in_host VARCHAR(60)`: 要检查的账户的主机名。

+   `in_user VARCHAR(32)`: 要检查的账户的用户名。

##### 返回值

一个 `ENUM('YES','NO')` 值。

##### 示例

```sql
mysql> SELECT sys.ps_is_account_enabled('localhost', 'root');
+------------------------------------------------+
| sys.ps_is_account_enabled('localhost', 'root') |
+------------------------------------------------+
| YES                                            |
+------------------------------------------------+
```
