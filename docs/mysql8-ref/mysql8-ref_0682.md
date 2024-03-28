# 12.3.9 字符集和排序规则分配示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-examples.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-examples.html)

以下示例展示了 MySQL 如何确定默认字符集和排序规则值。

**示例 1：表和列定义**

```sql
CREATE TABLE t1
(
    c1 CHAR(10) CHARACTER SET latin1 COLLATE latin1_german1_ci
) DEFAULT CHARACTER SET latin2 COLLATE latin2_bin;
```

这里有一个具有`latin1`字符集和`latin1_german1_ci`排序规则的列。定义是明确的，因此很直接。请注意，在`latin2`表中存储`latin1`列没有问题。

**示例 2：表和列定义**

```sql
CREATE TABLE t1
(
    c1 CHAR(10) CHARACTER SET latin1
) DEFAULT CHARACTER SET latin1 COLLATE latin1_danish_ci;
```

这次我们有一个具有`latin1`字符集和默认排序规则的列。虽然看起来很自然，但默认排序规则并不是从表级别获取的。相反，因为`latin1`的默认排序规则始终是`latin1_swedish_ci`，列`c1`的排序规则是`latin1_swedish_ci`（而不是`latin1_danish_ci`）。

**示例 3：表和列定义**

```sql
CREATE TABLE t1
(
    c1 CHAR(10)
) DEFAULT CHARACTER SET latin1 COLLATE latin1_danish_ci;
```

我们有一个具有默认字符集和默认排序规则的列。在这种情况下，MySQL 会检查表级别以确定列字符集和排序规则。因此，列`c1`的字符集是`latin1`，排序规则是`latin1_danish_ci`。

**示例 4：数据库、表和列定义**

```sql
CREATE DATABASE d1
    DEFAULT CHARACTER SET latin2 COLLATE latin2_czech_cs;
USE d1;
CREATE TABLE t1
(
    c1 CHAR(10)
);
```

创建一个没有指定字符集和排序规则的列。我们也没有在表级别指定字符集和排序规则。在这种情况下，MySQL 会检查数据库级别以确定表设置，然后成为列设置。因此，列`c1`的字符集是`latin2`，排序规则是`latin2_czech_cs`。
