# 18.2.3 MyISAM 表存储格式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-table-formats.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-table-formats.html)

18.2.3.1 静态（固定长度）表特性

18.2.3.2 动态表特性

18.2.3.3 压缩表特性

`MyISAM`支持三种不同的存储格式。其中两种，固定和动态格式，根据你使用的列类型自动选择。第三种，压缩格式，只能使用**myisampack**实用程序创建（参见第 6.6.6 节，“myisampack — 生成压缩的只读 MyISAM 表”）。

当你使用`CREATE TABLE`或`ALTER TABLE`创建没有`BLOB`或`TEXT`列的表时，可以使用`ROW_FORMAT`表选项强制表格式为`FIXED`或`DYNAMIC`。

查看第 15.1.20 节，“CREATE TABLE 语句”，了解`ROW_FORMAT`的信息。

你可以使用**myisamchk**的`--unpack`选项来解压缩`MyISAM`表；查看第 6.6.4 节，“myisamchk — MyISAM 表维护实用程序”获取更多信息。
