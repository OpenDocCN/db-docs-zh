# 2.10.3 使用 Perl DBI/DBD 接口时的问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/perl-support-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/perl-support-problems.html)

如果 Perl 报告找不到`../mysql/mysql.so`模块，则问题可能是 Perl 找不到`libmysqlclient.so`共享库的位置。您可以通过以下方法之一解决此问题：

+   将`libmysqlclient.so`复制到其他共享库所在的目录（可能是`/usr/lib`或`/lib`）。

+   修改用于编译`DBD::mysql`的`-L`选项，以反映`libmysqlclient.so`的实际位置。

+   在 Linux 上，您可以将包含`libmysqlclient.so`的目录路径添加到`/etc/ld.so.conf`文件中。

+   将包含`libmysqlclient.so`的目录路径添加到`LD_RUN_PATH`环境变量中。一些系统使用`LD_LIBRARY_PATH`代替。

请注意，如果链接器无法找到其他库，则可能还需要修改`-L`选项。例如，如果链接器无法找到`libc`，因为它在`/lib`中，而链接命令指定为`-L/usr/lib`，请将`-L`选项更改为`-L/lib`或将`-L/lib`添加到现有链接命令中。

如果从`DBD::mysql`获得以下错误，则您可能正在使用**gcc**（或使用用**gcc**编译的旧二进制文件）：

```sql
/usr/bin/perl: can't resolve symbol '__moddi3'
/usr/bin/perl: can't resolve symbol '__divdi3'
```

在构建`mysql.so`库时（编译 Perl 客户端时，请检查**make**的输出以查看`mysql.so`），在链接命令中添加`-L/usr/lib/gcc-lib/... -lgcc`。`-L`选项应指定系统上包含`libgcc.a`的目录路径。

此问题的另一个原因可能是 Perl 和 MySQL 都未使用**gcc**编译。在这种情况下，您可以通过使用**gcc**编译两者来解决不匹配问题。
