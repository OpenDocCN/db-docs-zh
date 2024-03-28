# 7.1.15 MySQL 服务器时区支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/time-zone-support.html`](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html)

本节描述了 MySQL 维护的时区设置，如何加载命名时间支持所需的系统表，如何跟进时区变更以及如何启用闰秒支持。

从 MySQL 8.0.19 开始，插入的日期时间值也支持时区偏移；请参见第 13.2.2 节，“DATE、DATETIME 和 TIMESTAMP 类型”，以获取更多信息。

有关复制设置中的时区设置信息，请参见第 19.5.1.14 节，“复制和系统函数”和第 19.5.1.33 节，“复制和时区”。

+   时区变量

+   填充时区表

+   时区变更跟进

+   时区闰秒支持

#### 时区变量

MySQL 服务器维护了几个时区设置：

+   服务器系统时区。服务器启动时，会尝试确定主机机器的时区并将其用于设置`system_time_zone`系统变量。

    要在 MySQL 服务器启动时明确指定系统时区，请在启动**mysqld**之前设置`TZ`环境变量。如果使用**mysqld_safe**启动服务器，则其`--timezone`选项提供了另一种设置系统时区的方式。`TZ`和`--timezone`的可接受值取决于系统。请查阅您的操作系统文档以查看哪些值是可接受的。

+   服务器当前时区。全局`time_zone`系统变量指示服务器当前操作的时区。初始`time_zone`值为`'SYSTEM'`，表示服务器时区与系统时区相同。

    注意

    如果设置为`SYSTEM`，则每个需要时区计算的 MySQL 函数调用都会进行系统库调用以确定当前系统时区。这个调用可能受到全局互斥锁的保护，导致争用。

    初始全局服务器时区值可以在启动时通过命令行上的`--default-time-zone`选项明确指定，或者可以在选项文件中使用以下行：

    ```sql
    default-time-zone='*timezone*'
    ```

    如果您拥有`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限），您可以使用以下语句在运行时设置全局服务器时区值：

    ```sql
    SET GLOBAL time_zone = *timezone*;
    ```

+   每个连接的客户端都有自己的会话时区设置，由会话`time_zone`变量给出。最初，会话变量从全局`time_zone`变量中获取其值，但客户端可以使用以下语句更改自己的时区：

    ```sql
    SET time_zone = *timezone*;
    ```

会话时区设置会影响对时区敏感的时间值的显示和存储。这包括由`NOW()`或`CURTIME()`等函数显示的值，以及存储在和从`TIMESTAMP`列中检索的值。`TIMESTAMP`列的值会在存储时从会话时区转换为 UTC，并在检索时从 UTC 转换为会话时区。

会话时区设置不会影响诸如`UTC_TIMESTAMP()`等函数显示的值，也不会影响`DATE`、`TIME`或`DATETIME`列中的值。这些数据类型中的值也不会以 UTC 存储；时区仅在从`TIMESTAMP`值转换时应用。如果您想要对`DATE`、`TIME`或`DATETIME`值进行区域特定的算术运算，将它们转换为 UTC，执行算术运算，然后再转换回来。

当前的全局和会话时区值可以这样检索：

```sql
SELECT @@GLOBAL.time_zone, @@SESSION.time_zone;
```

*`timezone`* 值可以以几种格式给出，大小写不敏感：

+   作为值`'SYSTEM'`，表示服务器时区与系统时区相同。

+   作为一个表示与 UTC 偏移的字符串，格式为`[*`H`*]*`H`*:*`MM`*`，前缀带有`+`或`-`，例如`'+10:00'`，`'-6:00'`或`'+05:30'`。对于小时值小于 10 的情况，可以选择性地使用前导零；在这种情况下，MySQL 在存储和检索值时会添加前导零。MySQL 将`'-00:00'`或`'-0:00'`转换为`'+00:00'`。

    在 MySQL 8.0.19 之前，此值必须在`'-12:59'`到`'+13:00'`的范围内；从 MySQL 8.0.19 开始，允许的范围是`'-13:59'`到`'+14:00'`，包括在内。

+   作为一个命名的时区，例如`'Europe/Helsinki'`，`'US/Eastern'`，`'MET'`或`'UTC'`。

    注意

    只有在`mysql`数据库中的时区信息表已创建并填充的情况下才能使用命名时区。否则，使用命名时区会导致错误：

    ```sql
    mysql> SET time_zone = 'UTC';
    ERROR 1298 (HY000): Unknown or incorrect time zone: 'UTC'
    ```

#### 填充时区表

`mysql`系统模式中存在几个表用于存储时区信息（参见第 7.3 节，“mysql 系统模式”）。MySQL 安装过程会创建时区表，但不会加载它们。要手动加载，请使用以下说明。

注意

加载时区信息不一定是一次性操作，因为信息偶尔会发生变化。当发生这种变化时，使用旧规则的应用程序会变得过时，您可能需要重新加载时区表以保持 MySQL 服务器使用的信息最新。请参阅保持时区更改最新。

如果您的系统有自己的 zoneinfo 数据库（描述时区的文件集），请使用**mysql_tzinfo_to_sql**程序加载时区表。这些系统的示例包括 Linux、macOS、FreeBSD 和 Solaris。这些文件的一个可能位置是`/usr/share/zoneinfo`目录。如果您的系统没有 zoneinfo 数据库，可以使用后面本节中描述的可下载包。

要从命令行加载时区表，请将 zoneinfo 目录路径名称传递给**mysql_tzinfo_to_sql**并将输出发送到**mysql**程序。例如：

```sql
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql
```

这里显示的**mysql**命令假定您使用具有修改`mysql`系统模式中表权限的帐户（如`root`）连接到服务器。根据需要调整连接参数。

**mysql_tzinfo_to_sql**读取系统的时区文件，并从中生成 SQL 语句。**mysql**处理这些语句以加载时区表。

**mysql_tzinfo_to_sql**也可用于加载单个时区文件或生成闰秒信息：

+   要加载与时区名称*`tz_name`*对应的单个时区文件*`tz_file`*，请像这样调用**mysql_tzinfo_to_sql**：

    ```sql
    mysql_tzinfo_to_sql *tz_file* *tz_name* | mysql -u root -p mysql
    ```

    使用这种方法，您必须为服务器需要了解的每个命名区域执行一个单独的命令来加载时区文件。

+   如果您的时区必须考虑闰秒，像这样初始化闰秒信息，其中*`tz_file`*是您的时区文件的名称：

    ```sql
    mysql_tzinfo_to_sql --leap *tz_file* | mysql -u root -p mysql
    ```

运行完**mysql_tzinfo_to_sql**后，重新启动服务器，以确保它不会继续使用任何先前缓存的时区数据。

如果您的系统没有 zoneinfo 数据库（例如 Windows），您可以使用一个包含 SQL 语句的软件包，可在 MySQL 开发者区下载：

```sql
https://dev.mysql.com/downloads/timezones.html
```

警告

如果您的系统有一个 zoneinfo 数据库，请不要使用可下载的时区包。而是使用**mysql_tzinfo_to_sql**实用程序。否则，可能会导致 MySQL 和系统上其他应用程序在处理日期时间时出现差异。

要使用已下载的 SQL 语句时区包，请解压缩它，然后将解压后的文件内容加载到时区表中：

```sql
mysql -u root -p mysql < *file_name*
```

然后重新启动服务器。

警告

不要使用包含`MyISAM`表的可下载时区包。这是为较旧的 MySQL 版本设计的。MySQL 现在使用`InnoDB`来存储时区表。尝试用`MyISAM`表替换它们会导致问题。

#### 保持与时区变更同步

当时区规则发生变化时，使用旧规则的应用程序会变得过时。为了保持最新，必须确保您的系统使用当前的时区信息。对于 MySQL，有多个因素需要考虑以保持最新：

+   操作系统时间会影响 MySQL 服务器用于时间的值，如果其时区设置为`SYSTEM`。确保您的操作系统使用最新的时区信息。对于大多数操作系统，最新的更新或服务包会为时间更改做准备。查看您操作系统供应商的网站，了解解决时间更改的更新。

+   如果您用一个使用与**mysqld**启动时不同的规则的版本替换系统的 `/etc/localtime` 时区文件，请重新启动**mysqld**以使用更新的规则。否则，**mysqld**可能不会注意到系统更改时间。

+   如果您在 MySQL 中使用命名时区，请确保 `mysql` 数据库中的时区表是最新的：

    +   如果您的系统有自己的 zoneinfo 数据库，请在 zoneinfo 数据库更新时重新加载 MySQL 时区表。

    +   对于没有自己的 zoneinfo 数据库的系统，请查看 MySQL 开发者区域以获取更新。当有新的更新可用时，下载并使用它来替换当前时区表的内容。

    有关两种方法的说明，请参见填充时区表。**mysqld** 缓存它查找的时区信息，因此在更新时区表后，重新启动**mysqld**以确保它不会继续提供过时的时区数据。

如果您不确定命名时区是否可用，无论是作为服务器的时区设置还是由设置自己时区的客户端使用，请检查您的时区表是否为空。以下查询确定包含时区名称的表是否有任何行：

```sql
mysql> SELECT COUNT(*) FROM mysql.time_zone_name;
+----------+
| COUNT(*) |
+----------+
|        0 |
+----------+
```

0 的计数表示表为空。在这种情况下，当前没有应用程序使用命名时区，您不需要更新表（除非您想启用命名时区支持）。大于零的计数表示表不为空，其内容可用于命名时区支持。在这种情况下，请确保重新加载您的时区表，以便使用命名时区的应用程序可以获得正确的查询结果。

要检查您的 MySQL 安装是否正确更新以适应夏令时规则的更改，请使用以下示例测试。该示例使用适用于 2007 年美国于 3 月 11 日凌晨 2 点发生的 DST 1 小时更改的值。

测试使用此查询：

```sql
SELECT
  CONVERT_TZ('2007-03-11 2:00:00','US/Eastern','US/Central') AS time1,
  CONVERT_TZ('2007-03-11 3:00:00','US/Eastern','US/Central') AS time2;
```

两个时间值表示 DST 更改发生的时间，使用命名时区需要使用时区表。期望的结果是两个查询返回相同的结果（输入时间转换为 'US/Central' 时区中的等效值）。

在更新时区表之前，您会看到类似于这样的错误结果：

```sql
+---------------------+---------------------+
| time1               | time2               |
+---------------------+---------------------+
| 2007-03-11 01:00:00 | 2007-03-11 02:00:00 |
+---------------------+---------------------+
```

更新表后，您应该看到正确的结果：

```sql
+---------------------+---------------------+
| time1               | time2               |
+---------------------+---------------------+
| 2007-03-11 01:00:00 | 2007-03-11 01:00:00 |
+---------------------+---------------------+
```

#### 时区闰秒支持

跳秒值返回的时间部分以`:59:59`结尾。这意味着在跳秒期间，诸如`NOW()`之类的函数可能在跳秒期间的两三秒内返回相同的值。仍然正确的是，时间部分以`:59:60`或`:59:61`结尾的文字时间值被视为无效。

如果需要搜索跳秒前一秒的`TIMESTAMP`值，如果使用与`'*`YYYY-MM-DD hh:mm:ss`*'`值进行比较，可能会得到异常结果。以下示例演示了这一点。它将会话时区更改为 UTC，因此内部`TIMESTAMP`值（在 UTC 中）和显示值（应用了时区校正的值）之间没有区别。

```sql
mysql> CREATE TABLE t1 (
         a INT,
         ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         PRIMARY KEY (ts)
       );
Query OK, 0 rows affected (0.01 sec)

mysql> -- change to UTC
mysql> SET time_zone = '+00:00';
Query OK, 0 rows affected (0.00 sec)

mysql> -- Simulate NOW() = '2008-12-31 23:59:59'
mysql> SET timestamp = 1230767999;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 (a) VALUES (1);
Query OK, 1 row affected (0.00 sec)

mysql> -- Simulate NOW() = '2008-12-31 23:59:60'
mysql> SET timestamp = 1230768000;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 (a) VALUES (2);
Query OK, 1 row affected (0.00 sec)

mysql> -- values differ internally but display the same
mysql> SELECT a, ts, UNIX_TIMESTAMP(ts) FROM t1;
+------+---------------------+--------------------+
| a    | ts                  | UNIX_TIMESTAMP(ts) |
+------+---------------------+--------------------+
|    1 | 2008-12-31 23:59:59 |         1230767999 |
|    2 | 2008-12-31 23:59:59 |         1230768000 |
+------+---------------------+--------------------+
2 rows in set (0.00 sec)

mysql> -- only the non-leap value matches
mysql> SELECT * FROM t1 WHERE ts = '2008-12-31 23:59:59';
+------+---------------------+
| a    | ts                  |
+------+---------------------+
|    1 | 2008-12-31 23:59:59 |
+------+---------------------+
1 row in set (0.00 sec)

mysql> -- the leap value with seconds=60 is invalid
mysql> SELECT * FROM t1 WHERE ts = '2008-12-31 23:59:60';
Empty set, 2 warnings (0.00 sec)
```

为了解决这个问题，您可以使用基于实际存储在列中的 UTC 值的比较，其中应用了跳秒校正：

```sql
mysql> -- selecting using UNIX_TIMESTAMP value return leap value
mysql> SELECT * FROM t1 WHERE UNIX_TIMESTAMP(ts) = 1230768000;
+------+---------------------+
| a    | ts                  |
+------+---------------------+
|    2 | 2008-12-31 23:59:59 |
+------+---------------------+
1 row in set (0.00 sec)
```
