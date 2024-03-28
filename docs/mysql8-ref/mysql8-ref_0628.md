> 原文：[`dev.mysql.com/doc/refman/8.0/en/symbolic-links-to-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/symbolic-links-to-tables.html)

#### 10.12.2.2 在 Unix 上使用符号链接的 MyISAM 表

注意

此处描述的符号链接支持，以及控制它的`--symbolic-links`选项已被弃用；预计在未来的 MySQL 版本中将删除这些内容。此外，默认情况下该选项已禁用。

仅对 `MyISAM` 表完全支持符号链接。对于其他存储引擎表使用的文件，如果尝试使用符号链接可能会出现奇怪的问题。对于 `InnoDB` 表，请使用 Section 17.6.1.2, “Creating Tables Externally”中解释的替代技术。

在没有完全运行`realpath()`调用的系统上不要对表进行符号链接。 (Linux 和 Solaris 支持`realpath()`). 要确定您的系统是否支持符号链接，请使用以下语句检查`have_symlink`系统变量的值：

```sql
SHOW VARIABLES LIKE 'have_symlink';
```

以下是 `MyISAM` 表的符号链接处理方式：

+   在数据目录中，您始终有数据（`.MYD`）文件和索引（`.MYI`）文件。数据文件和索引文件可以移动到其他位置，并在数据目录中用符号链接替换。

+   你可以将数据文件和索引文件分别链接到不同的目录。

+   要指示正在运行的 MySQL 服务器执行符号链接操作，请使用`CREATE TABLE`中的`DATA DIRECTORY`和`INDEX DIRECTORY`选项。参见 Section 15.1.20, “CREATE TABLE Statement”。或者，如果**mysqld**没有运行，可以使用命令行中的**ln -s**手动完成符号链接。

    注意

    与`DATA DIRECTORY`和`INDEX DIRECTORY`选项中的路径可能不包括 MySQL `data` 目录。 (Bug #32167)

+   **myisamchk**不会用数据文件或索引文件替换符号链接。它直接在符号链接指向的文件上工作。任何临时文件都将在数据文件或索引文件所在的目录中创建。对于`ALTER TABLE`、`OPTIMIZE TABLE`和`REPAIR TABLE`语句也是如此。

+   注意

    当您删除一个使用符号链接的表时，*符号链接和符号链接指向的文件都会被删除*。这是一个极好的理由*不*以`root`操作系统用户身份运行**mysqld**或允许操作系统用户对 MySQL 数据库目录具有写访问权限。

+   如果您使用`ALTER TABLE ... RENAME`或`RENAME TABLE`重命名一个表，并且不将表移动到另一个数据库，则数据库目录中的符号链接将被重命名为新名称，并相应地重命名数据文件和索引文件。

+   如果您使用`ALTER TABLE ... RENAME`或`RENAME TABLE`将表移动到另一个数据库，表将移动到另一个数据库目录。如果表名发生更改，则新数据库目录中的符号链接将被重命名为新名称，并相应地重命名数据文件和索引文件。

+   如果您不使用符号链接，请使用`--skip-symbolic-links`选项启动**mysqld**，以确保没有人可以使用**mysqld**删除或重命名数据目录之外的文件。

这些表符号链接操作不受支持：

+   `ALTER TABLE`会忽略`DATA DIRECTORY`和`INDEX DIRECTORY`表选项。
