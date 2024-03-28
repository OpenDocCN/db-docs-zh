# 2.10.2 在 Windows 上安装 ActiveState Perl

> 原文：[`dev.mysql.com/doc/refman/8.0/en/activestate-perl.html`](https://dev.mysql.com/doc/refman/8.0/en/activestate-perl.html)

在 Windows 上，您应该按照以下步骤使用 ActiveState Perl 安装 MySQL `DBD`模块：

1.  从[`www.activestate.com/Products/ActivePerl/`](http://www.activestate.com/Products/ActivePerl/)获取 ActiveState Perl 并安装。

1.  打开控制台窗口。

1.  如果需要，设置`HTTP_proxy`变量。例如，您可以尝试这样的设置：

    ```sql
    C:\> set HTTP_proxy=my.proxy.com:3128
    ```

1.  启动 PPM 程序：

    ```sql
    C:\> C:\perl\bin\ppm.pl
    ```

1.  如果之前没有安装过，安装`DBI`：

    ```sql
    ppm> install DBI
    ```

1.  如果成功，运行以下命令：

    ```sql
    ppm> install DBD-mysql
    ```

这个步骤适用于 ActiveState Perl 5.6 或更高版本。

如果无法使该步骤生效，应安装 ODBC 驱动程序，然后通过 ODBC 连接到 MySQL 服务器：

```sql
use DBI;
$dbh= DBI->connect("DBI:ODBC:$dsn",$user,$password) ||
  die "Got error $DBI::errstr when connecting to $dsn\n";
```
