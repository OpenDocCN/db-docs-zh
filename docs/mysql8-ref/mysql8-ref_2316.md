> 原文：[`dev.mysql.com/doc/refman/8.0/en/file-permissions.html`](https://dev.mysql.com/doc/refman/8.0/en/file-permissions.html)

#### B.3.3.1 文件权限问题

如果您在文件权限方面遇到问题，可能是在**mysqld**启动时`UMASK`或`UMASK_DIR`环境变量设置不正确。例如，当您创建表时，**mysqld**可能会发出以下错误消息：

```sql
ERROR: Can't find file: 'path/with/*file_name*' (Errcode: 13)
```

默认的`UMASK`和`UMASK_DIR`值分别为`0640`和`0750`。**mysqld**假定如果`UMASK`或`UMASK_DIR`的值以零开头，则为八进制。例如，设置`UMASK=0600`等同于`UMASK=384`，因为八进制的 0600 等于十进制的 384。

假设您使用**mysqld_safe**启动**mysqld**，请按以下方式更改默认的`UMASK`值：

```sql
UMASK=384  # = 600 in octal
export UMASK
mysqld_safe &
```

注意

如果您使用**mysqld_safe**启动**mysqld**，错误日志文件是个例外，因为**mysqld_safe**不遵守`UMASK`：如果在启动**mysqld**之前错误日志文件不存在，**mysqld_safe**可能会创建错误日志文件，并且**mysqld_safe**使用严格值为`0137`的 umask。如果这不合适，请在执行**mysqld_safe**之前手动创建所需访问模式的错误文件。

默认情况下，**mysqld**以`0750`的访问权限值创建数据库目录。要修改此行为，请设置`UMASK_DIR`变量。如果设置了其值，新目录将以组合的`UMASK`和`UMASK_DIR`值创建。例如，要给所有新目录提供组访问权限，请按以下方式启动**mysqld_safe**：

```sql
UMASK_DIR=504  # = 770 in octal
export UMASK_DIR
mysqld_safe &
```

更多详情，请参阅第 6.9 节“环境变量”。
