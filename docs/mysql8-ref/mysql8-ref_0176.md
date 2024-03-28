# 6.4.4 mysql_tzinfo_to_sql — 加载时区表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-tzinfo-to-sql.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-tzinfo-to-sql.html)

**mysql_tzinfo_to_sql**程序加载`mysql`数据库中的时区表。它用于具有区域信息数据库（描述时区的文件集）的系统。这些系统的示例包括 Linux、FreeBSD、Solaris 和 macOS。这些文件的一个可能位置是`/usr/share/zoneinfo`目录（Solaris 上为`/usr/share/lib/zoneinfo`）。如果您的系统没有区域信息数据库，可以使用第 7.1.15 节“MySQL 服务器时区支持”中描述的可下载软件包。

**mysql_tzinfo_to_sql**可以以几种方式调用：

```sql
mysql_tzinfo_to_sql *tz_dir*
mysql_tzinfo_to_sql *tz_file tz_name*
mysql_tzinfo_to_sql --leap *tz_file*
```

对于第一种调用语法，将区域信息目录路径名称传递给**mysql_tzinfo_to_sql**，并将输出发送到**mysql**程序。例如：

```sql
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql
```

**mysql_tzinfo_to_sql**读取系统的时区文件，并从中生成 SQL 语句。**mysql**处理这些语句以加载时区表。

第二种语法导致**mysql_tzinfo_to_sql**加载一个对应于时区名称*`tz_name`*的单个时区文件*`tz_file`*：

```sql
mysql_tzinfo_to_sql *tz_file* *tz_name* | mysql -u root mysql
```

如果您的时区需要考虑闰秒，使用第三种语法调用**mysql_tzinfo_to_sql**，该语法初始化闰秒信息。*`tz_file`*是您的时区文件的名称：

```sql
mysql_tzinfo_to_sql --leap *tz_file* | mysql -u root mysql
```

运行**mysql_tzinfo_to_sql**后，最好重新启动服务器，以便它不继续使用任何先前缓存的时区数据。
