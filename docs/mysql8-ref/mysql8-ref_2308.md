> 原文：[`dev.mysql.com/doc/refman/8.0/en/cannot-create.html`](https://dev.mysql.com/doc/refman/8.0/en/cannot-create.html)

#### B.3.2.11 无法创建/写入文件

如果对某些查询出现以下类型的错误，意味着 MySQL 无法在临时目录中为结果集创建临时文件：

```sql
Can't create/write to file '\\sqla3fe_0.ism'.
```

上述错误是 Windows 的典型消息；Unix 的消息类似。

一种解决方法是使用 `--tmpdir` 选项启动 **mysqld**，或将该选项添加到选项文件的 `[mysqld]` 部分。例如，要指定目录为 `C:\temp`，请使用以下行：

```sql
[mysqld]
tmpdir=C:/temp
```

`C:\temp` 目录必须存在并具有足够的空间供 MySQL 服务器写入。参见 Section 6.2.2.2, “使用选项文件”。

这个错误的另一个原因可能是权限问题。确保 MySQL 服务器可以写入 `tmpdir` 目录。

还要检查使用 **perror** 得到的错误代码。服务器无法写入表的一个原因是文件系统已满：

```sql
$> perror 28
OS error code  28:  No space left on device
```

如果在启动过程中出现以下类型的错误，表明用于存储数据文件的文件系统或目录受到写保护。只要写错误是针对测试文件的，那么这个错误并不严重，可以安全地忽略。

```sql
Can't create test file /usr/local/mysql/data/master.lower-test
```
