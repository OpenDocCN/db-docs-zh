> 原文：[`dev.mysql.com/doc/refman/8.0/en/password-too-long.html`](https://dev.mysql.com/doc/refman/8.0/en/password-too-long.html)

#### B.3.2.4 交互式输入密码失败

当使用`--password`或`-p`选项调用 MySQL 客户端程序时，会提示输入密码，但没有后续密码值：

```sql
$> mysql -u *user_name* -p
Enter password:
```

在某些系统上，当在选项文件或命令行中指定密码时可以正常工作，但在交互式输入`输入密码：`提示时却无法正常工作。这是因为系统提供的用于读取密码的库将密码值限制在少量字符（通常为八个字符）内。这是系统库的问题，而不是 MySQL 的问题。为了解决这个问题，将 MySQL 密码更改为八个或更少字符的值，或将密码放入选项文件中。
