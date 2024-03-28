> 原文：[`dev.mysql.com/doc/refman/8.0/en/ansi-diff-update.html`](https://dev.mysql.com/doc/refman/8.0/en/ansi-diff-update.html)

#### 1.6.2.2 UPDATE 差异

如果在表达式中访问要更新的表中的列，`UPDATE` 会使用列的当前值。以下语句中的第二个赋值将`col2`设置为当前（更新后）的`col1`值，而不是原始的`col1`值。结果是`col1`和`col2`具有相同的值。这种行为与标准 SQL 不同。

```sql
UPDATE t1 SET col1 = col1 + 1, col2 = col1;
```
