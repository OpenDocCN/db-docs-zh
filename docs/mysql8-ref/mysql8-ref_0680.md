# 12.3.7 国家字符集

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-national.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-national.html)

标准 SQL 将`NCHAR`或`NATIONAL CHAR`定义为指示`CHAR`列应使用某个预定义字符集的方法。MySQL 使用`utf8`作为这个预定义字符集。例如，以下数据类型声明是等效的：

```sql
CHAR(10) CHARACTER SET utf8
NATIONAL CHARACTER(10)
NCHAR(10)
```

此外还有这些：

```sql
VARCHAR(10) CHARACTER SET utf8
NATIONAL VARCHAR(10)
NVARCHAR(10)
NCHAR VARCHAR(10)
NATIONAL CHARACTER VARYING(10)
NATIONAL CHAR VARYING(10)
```

您可以使用`N'*`literal`*'`（或`n'*`literal`*'`）来创建一个国家字符集中的字符串。以下语句是等效的：

```sql
SELECT N'some text';
SELECT n'some text';
SELECT _utf8'some text';
```

MySQL 8.0 将国家字符集解释为`utf8mb3`，这种方式现在已经被弃用。因此，使用`NATIONAL CHARACTER`或其同义词之一来定义数据库、表或列的字符集会引发类似于以下警告：

```sql
NATIONAL/NCHAR/NVARCHAR implies the character set UTF8MB3, which will be
replaced by UTF8MB4 in a future release. Please consider using CHAR(x) CHARACTER
SET UTF8MB4 in order to be unambiguous.
```
