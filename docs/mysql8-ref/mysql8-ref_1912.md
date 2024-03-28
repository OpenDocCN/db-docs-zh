# 28.3.17 INFORMATION_SCHEMA KEYWORDS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-keywords-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-keywords-table.html)

`KEYWORDS` 表列出了 MySQL 认为是关键字的单词，并针对每个单词指示它是否是保留的。在某些情况下，保留关键字可能需要特殊处理，例如在用作标识符时需要特殊引用（参见第 11.3 节，“关键字和保留字”）。这个表为应用程序提供了 MySQL 关键字信息的运行时来源。

在 MySQL 8.0.13 之前，选择`KEYWORDS`表而没有选择默认数据库会产生错误。（Bug #90160, Bug #27729859）

`KEYWORDS` 表具有以下列：

+   `WORD`

    关键字。

+   `RESERVED`

    一个整数，指示关键字是保留的（1）还是非保留的（0）。

这些查询分别列出所有关键字、所有保留关键字和所有非保留关键字：

```sql
SELECT * FROM INFORMATION_SCHEMA.KEYWORDS;
SELECT WORD FROM INFORMATION_SCHEMA.KEYWORDS WHERE RESERVED = 1;
SELECT WORD FROM INFORMATION_SCHEMA.KEYWORDS WHERE RESERVED = 0;
```

后两个查询等同于：

```sql
SELECT WORD FROM INFORMATION_SCHEMA.KEYWORDS WHERE RESERVED;
SELECT WORD FROM INFORMATION_SCHEMA.KEYWORDS WHERE NOT RESERVED;
```

如果你从源代码构建 MySQL，构建过程会生成一个`keyword_list.h`头文件，其中包含关键字及其保留状态的数组。该文件可以在构建目录下的`sql`目录中找到。对于需要关键字列表的应用程序，这个文件可能很有用。
