# 7.1.1 配置服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/server-configuration.html)

MySQL 服务器**mysqld**具有许多在启动时可以设置以配置其操作的命令选项和系统变量。要确定服务器使用的默认命令选项和系统变量值，请执行此命令：

```sql
$> mysqld --verbose --help
```

该命令生成所有**mysqld**选项和可配置系统变量的列表。其输出包括默认选项和变量值，并且看起来类似于这样：

```sql
abort-slave-event-count           0
allow-suspicious-udfs             FALSE
archive                           ON
auto-increment-increment          1
auto-increment-offset             1
autocommit                        TRUE
automatic-sp-privileges           TRUE
avoid-temporal-upgrade            FALSE
back-log                          80
basedir                           /home/jon/bin/mysql-8.0/
...
tmpdir                            /tmp
transaction-alloc-block-size      8192
transaction-isolation             REPEATABLE-READ
transaction-prealloc-size         4096
transaction-read-only             FALSE
transaction-write-set-extraction  XXHASH64
updatable-views-with-limit        YES
validate-user-plugins             TRUE
verbose                           TRUE
wait-timeout                      28800
```

要查看服务器运行时实际使用的当前系统变量值，请连接到服务器并执行此语句：

```sql
mysql> SHOW VARIABLES;
```

要查看运行中服务器的一些统计和状态指标，请执行此语句：

```sql
mysql> SHOW STATUS;
```

还可以使用**mysqladmin**命令获取系统变量和状态信息：

```sql
$> mysqladmin variables
$> mysqladmin extended-status
```

要获取所有命令选项、系统变量和状态变量的详细描述，请参阅这些章节：

+   Section 7.1.7, “服务器命令选项”

+   Section 7.1.8, “服务器系统变量”

+   Section 7.1.10, “服务器状态变量”

更详细的监控信息可从性能模式中获得；请参阅第二十九章，*MySQL 性能模式*。此外，MySQL `sys`模式是一组对象，提供了方便访问性能模式收集的数据；请参阅第三十章，*MySQL sys 模式*。

如果在命令行上为**mysqld**或**mysqld_safe**指定选项，则仅在服务器的该调用中有效。要使该选项在每次服务器运行时都生效，请将其放入选项文件中。参见 Section 6.2.2.2, “使用选项文件”。
