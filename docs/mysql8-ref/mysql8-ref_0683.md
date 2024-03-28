# 12.3.10 与其他 DBMS 的兼容性

> [`dev.mysql.com/doc/refman/8.0/en/charset-compatibility.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-compatibility.html)

对于 MaxDB 兼容性，这两个语句是相同的：

```sql
CREATE TABLE t1 (f1 CHAR(*N*) UNICODE);
CREATE TABLE t1 (f1 CHAR(*N*) CHARACTER SET ucs2);
```

`UNICODE`属性和`ucs2`字符集在 MySQL 8.0.28 中已被弃用。
