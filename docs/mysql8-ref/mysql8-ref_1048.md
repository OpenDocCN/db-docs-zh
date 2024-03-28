> 原文：[`dev.mysql.com/doc/refman/8.0/en/rename-user.html`](https://dev.mysql.com/doc/refman/8.0/en/rename-user.html)

#### 15.7.1.7 `RENAME USER` 语句

```sql
RENAME USER *old_user* TO *new_user*
    [, *old_user* TO *new_user*] ...
```

`RENAME USER` 语句重命名现有的 MySQL 帐户。对于不存在的旧帐户或已存在的新帐户，将出现错误。

要使用`RENAME USER`，您必须具有全局`CREATE USER`特权，或者对`mysql`系统模式具有`UPDATE`特权。当启用`read_only`系统变量时，`RENAME USER` 还需要`CONNECTION_ADMIN`特权（或已弃用的`SUPER`特权）。

截至 MySQL 8.0.22 版，如果要重命名的任何帐户被命名为任何存储对象的`DEFINER`属性，则`RENAME USER` 将失败并显示错误。（也就是说，如果重命名帐户会导致存储对象变成孤立状态，则该语句将失败。）要执行操作，您必须具有`SET_USER_ID`特权；在这种情况下，该语句将成功并显示警告，而不是失败并显示错误。有关更多信息，包括如何识别哪些对象将给定帐户命名为`DEFINER`属性，请参见孤立存储对象。

每个帐户名使用第 8.2.4 节，“指定帐户名”中描述的格式。例如：

```sql
RENAME USER 'jeffrey'@'localhost' TO 'jeff'@'127.0.0.1';
```

账户名的主机名部分，如果省略，默认为`'%'`。

`RENAME USER` 导致旧用户持有的特权变为新用户持有的特权。然而，`RENAME USER` 不会自动删除或使旧用户创建的数据库或其中的对象失效。这包括`DEFINER`属性命名旧用户的存储过程或视图。如果在定义者安全上下文中执行这些对象，访问这些对象可能会产生错误。（有关安全上下文的信息，请参见第 27.6 节，“存储对象访问控制”。）

特权更改将按照第 8.2.13 节，“特权更改生效时间”中指示的方式生效。
