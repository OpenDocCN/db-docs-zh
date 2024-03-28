> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-execute-prepared-stmt.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-execute-prepared-stmt.html)

#### 30.4.4.3 `execute_prepared_stmt()` 过程

给定一个作为字符串的 SQL 语句，将其作为准备好的语句执行。执行后，准备好的语句将被取消分配，因此不会被重用。因此，此过程主要用于一次性执行动态语句。

此过程使用`sys_execute_prepared_stmt`作为准备好的语句名称。如果在调用过程时存在该语句名称，则其先前的内容将被销毁。

##### 参数

+   `in_query LONGTEXT CHARACTER SET utf8mb3`：要执行的语句字符串。

##### 配置选项

`execute_prepared_stmt()` Procedure") 操作可以使用以下配置选项或其对应的用户定义变量进行修改（参见 Section 30.4.2.1, “The sys_config Table”）：

+   `debug`，`@sys.debug`

    如果此选项为`ON`，则生成调试输出。默认为`OFF`。

##### 示例

```sql
mysql> CALL sys.execute_prepared_stmt('SELECT COUNT(*) FROM mysql.user');
+----------+
| COUNT(*) |
+----------+
|       15 |
+----------+
```
