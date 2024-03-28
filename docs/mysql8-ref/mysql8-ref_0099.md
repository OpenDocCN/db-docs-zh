# 2.10.1 在 Unix 上安装 Perl

> 原文：[`dev.mysql.com/doc/refman/8.0/en/perl-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/perl-installation.html)

MySQL Perl 支持要求您已安装了 MySQL 客户端编程支持（库和头文件）。大多数安装方法都会安装必要的文件。如果您在 Linux 上从 RPM 文件安装 MySQL，请确保也安装了开发者 RPM。客户端程序在客户端 RPM 中，但客户端编程支持在开发者 RPM 中。

您需要用于 Perl 支持的文件可以从 CPAN（Comprehensive Perl Archive Network）获取，网址为[`search.cpan.org`](http://search.cpan.org)。

在 Unix 上安装 Perl 模块的最简单方法是使用`CPAN`模块。例如：

```sql
$> perl -MCPAN -e shell
cpan> install DBI
cpan> install DBD::mysql
```

`DBD::mysql`安装运行了一系列测试。这些测试尝试使用默认用户名和密码连接到本地 MySQL 服务器。（默认用户名是 Unix 上的登录名，Windows 上是`ODBC`。默认密码是“no password”。）如果您无法使用这些值连接到服务器（例如，如果您的帐户有密码），测试将失败。您可以使用`force install DBD::mysql`来忽略失败的测试。

`DBI`需要`Data::Dumper`模块。如果没有安装，您应该在安装`DBI`之前安装它。

也可以以压缩的**tar**存档形式下载模块分发，并手动构建模块。例如，要解压和构建一个 DBI 分发，可以使用以下过程：

1.  解压分发到当前目录：

    ```sql
    $> gunzip < DBI-*VERSION*.tar.gz | tar xvf -
    ```

    这个命令会创建一个名为`DBI-*`VERSION`*`的目录。

1.  将位置更改为解压分发的顶级目录：

    ```sql
    $> cd DBI-*VERSION*
    ```

1.  构建分发并编译所有内容：

    ```sql
    $> perl Makefile.PL
    $> make
    $> make test
    $> make install
    ```

**make test**命令很重要，因为它验证模块是否正常工作。请注意，在`DBD::mysql`安装期间运行该命令以执行接口代码时，MySQL 服务器必须正在运行，否则测试将失败。

重新构建和重新安装`DBD::mysql`分发是一个好主意，每当你安装新版本的 MySQL 时。这确保了最新版本的 MySQL 客户端库被正确安装。

如果您没有权限在系统目录中安装 Perl 模块，或者想要安装本地 Perl 模块，下面的参考可能会有用：[`learn.perl.org/faq/perlfaq8.html#How-do-I-keep-my-own-module-library-directory-`](http://learn.perl.org/faq/perlfaq8.html#How-do-I-keep-my-own-module-library-directory-)
