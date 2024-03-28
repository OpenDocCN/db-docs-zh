# 12.8.3 字符集和排序兼容性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collation-compatibility.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-compatibility.html)

每个字符集都有一个或多个排序规则，但每个排序规则只与一个字符集相关联。因此，以下语句会导致错误消息，因为`latin2_bin`排序规则与`latin1`字符集不兼容：

```sql
mysql> SELECT _latin1 'x' COLLATE latin2_bin;
ERROR 1253 (42000): COLLATION 'latin2_bin' is not valid
for CHARACTER SET 'latin1'
```
