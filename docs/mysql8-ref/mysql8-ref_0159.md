> [`dev.mysql.com/doc/refman/8.0/en/option-defaults-equals.html`](https://dev.mysql.com/doc/refman/8.0/en/option-defaults-equals.html)

#### 6.2.2.6 选项默认值、期望值的选项和等号

按照惯例，分配值的长形式选项使用等号（`=`）符号编写，就像这样：

```sql
mysql --host=tonfisk --user=jon
```

对于需要值的选项（即没有默认值的选项），等号是不需要的，因此以下内容也是有效的：

```sql
mysql --host tonfisk --user jon
```

在这两种情况下，**mysql**客户端尝试连接到名为“tonfisk”的主机上运行的 MySQL 服务器，使用用户名“jon”。

由于这种行为，当某个期望值的选项没有提供值时，有时会出现问题。考虑以下示例，用户连接到运行在主机`tonfisk`上的 MySQL 服务器，用户名为`jon`：

```sql
$> mysql --host 85.224.35.45 --user jon
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 8.0.36 Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SELECT CURRENT_USER();
+----------------+
| CURRENT_USER() |
+----------------+
| jon@%          |
+----------------+
1 row in set (0.00 sec)
```

省略其中一个选项所需的值会导致错误，如下所示：

```sql
$> mysql --host 85.224.35.45 --user
mysql: option '--user' requires an argument
```

在这种情况下，**mysql**无法找到跟在`--user`选项后面的值，因为在命令行中它后面没有内容。然而，如果你省略了*不是*最后一个要使用的选项的值，你会得到一个不同的错误，可能不是你期望的：

```sql
$> mysql --host --user jon
ERROR 2005 (HY000): Unknown MySQL server host '--user' (1)
```

因为**mysql**假定在命令行中跟在`--host`后面的任何字符串都是主机名，`--host` `--user`被解释为`--host=--user`，客户端尝试连���到名为`--user`的 MySQL 服务器。

具有默认值的选项在分配值时总是需要等号；如果不这样做会导致错误。例如，MySQL 服务器的`--log-error`选项具有默认值`*`host_name`*.err`，其中*`host_name`*是 MySQL 正在运行的主机的名称。假设你正在一台名为“tonfisk”的计算机上运行 MySQL，并考虑以下对**mysqld_safe**的调用：

```sql
$> mysqld_safe &
[1] 11699
$> 080112 12:53:40 mysqld_safe Logging to '/usr/local/mysql/var/tonfisk.err'.
080112 12:53:40 mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/var
$>
```

关闭服务器后，按以下方式重新启动：

```sql
$> mysqld_safe --log-error &
[1] 11699
$> 080112 12:53:40 mysqld_safe Logging to '/usr/local/mysql/var/tonfisk.err'.
080112 12:53:40 mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/var
$>
```

结果是相同的，因为`--log-error`后面没有跟任何其他内容，它提供了自己的默认值。（`&`字符告诉操作系统在后台运行 MySQL；MySQL 本身会忽略它。）现在假设你希望将错误日志记录到名为`my-errors.err`的文件中。你可能尝试使用`--log-error my-errors`来启动服务器，但这并没有产生预期的效果，如下所示：

```sql
$> mysqld_safe --log-error my-errors &
[1] 31357
$> 080111 22:53:31 mysqld_safe Logging to '/usr/local/mysql/var/tonfisk.err'.
080111 22:53:32 mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/var
080111 22:53:34 mysqld_safe mysqld from pid file /usr/local/mysql/var/tonfisk.pid ended

[1]+  Done                    ./mysqld_safe --log-error my-errors
```

服务器尝试使用`/usr/local/mysql/var/tonfisk.err`作为错误日志启动，但随后关闭。检查这个文件的最后几行显示了原因：

```sql
$> tail /usr/local/mysql/var/tonfisk.err
2013-09-24T15:36:22.278034Z 0 [ERROR] Too many arguments (first extra is 'my-errors').
2013-09-24T15:36:22.278059Z 0 [Note] Use --verbose --help to get a list of available options!
2013-09-24T15:36:22.278076Z 0 [ERROR] Aborting
2013-09-24T15:36:22.279704Z 0 [Note] InnoDB: Starting shutdown...
2013-09-24T15:36:23.777471Z 0 [Note] InnoDB: Shutdown completed; log sequence number 2319086
2013-09-24T15:36:23.780134Z 0 [Note] mysqld: Shutdown complete
```

因为`--log-error`选项提供了默认值，你必须使用等号来分配不同的值，如下所示：

```sql
$> mysqld_safe --log-error=my-errors &
[1] 31437
$> 080111 22:54:15 mysqld_safe Logging to '/usr/local/mysql/var/my-errors.err'.
080111 22:54:15 mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/var

$>
```

现在服务器已成功启动，并将错误记录到文件`/usr/local/mysql/var/my-errors.err`中。

在选项文件中指定选项值时可能会出现类似的问题。例如，考虑一个包含以下内容的`my.cnf`文件：

```sql
[mysql]

host
user
```

当**mysql**客户端读取这个文件时，这些条目会被解析为`--host` `--user` 或 `--host=--user`，结果如下所示：

```sql
$> mysql
ERROR 2005 (HY000): Unknown MySQL server host '--user' (1)
```

然而，在选项文件中，并不会默认使用等号。假设`my.cnf`文件如下所示：

```sql
[mysql]

user jon
```

在这种情况下尝试启动**mysql**会导致不同的错误：

```sql
$> mysql
mysql: unknown option '--user jon'
```

如果你在选项文件中写入`host tonfisk`而不是`host=tonfisk`，就会发生类似的错误。你必须使用等号：

```sql
[mysql]

user=jon
```

现在登录尝试成功了：

```sql
$> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 8.0.36 Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SELECT USER();
+---------------+
| USER()        |
+---------------+
| jon@localhost |
+---------------+
1 row in set (0.00 sec)
```

这与在命令行中的行为不同，命令行中不需要等号：

```sql
$> mysql --user jon --host tonfisk
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 8.0.36 Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SELECT USER();
+---------------+
| USER()        |
+---------------+
| jon@tonfisk   |
+---------------+
1 row in set (0.00 sec)
```

在选项文件中指定需要值但没有值的选项会导致服务器出现错误而中止。
