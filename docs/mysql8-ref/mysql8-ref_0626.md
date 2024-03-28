# 10.12.2 使用符号链接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/symbolic-links.html`](https://dev.mysql.com/doc/refman/8.0/en/symbolic-links.html)

10.12.2.1 在 Unix 上使用符号链接处理数据库

10.12.2.2 在 Unix 上使用符号链接处理 MyISAM 表

10.12.2.3 在 Windows 上使用符号链接处理数据库

您可以将数据库或表从数据库目录移动到其他位置，并用指向新位置的符号链接替换它们。例如，您可能想要这样做，将数据库移动到具有更多可用空间的文件系统，或者通过将表分布到不同的磁盘上来提高系统速度。

对于`InnoDB`表，使用`CREATE TABLE`语句的`DATA DIRECTORY`子句，而不是符号链接，如 Section 17.6.1.2，“外部创建表”中所解释的那样。这个新功能是一种受支持的跨平台技术。

推荐的方法是将整个数据库目录创建符号链接到不同的磁盘上。只有在万不得已的情况下才将`MyISAM`表创建符号链接。

要确定数据目录的位置，请使用以下语句：

```sql
SHOW VARIABLES LIKE 'datadir';
```
