# 18.4.2 CSV 限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/se-csv-limitations.html`](https://dev.mysql.com/doc/refman/8.0/en/se-csv-limitations.html)

`CSV`存储引擎不支持索引。

`CSV`存储引擎不支持分区。

使用`CSV`存储引擎创建的所有表必须在所有列上具有`NOT NULL`属性。
