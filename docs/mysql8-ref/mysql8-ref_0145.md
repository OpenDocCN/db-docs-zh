# 5.6.7 在两个键上进行搜索

> 原文：[`dev.mysql.com/doc/refman/8.0/en/searching-on-two-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/searching-on-two-keys.html)

使用单个键的`OR`是经过良好优化的，处理`AND`也是如此。

唯一棘手的情况是在两个不同键上结合使用`OR`进行搜索：

```sql
SELECT field1_index, field2_index FROM test_table
WHERE field1_index = '1' OR  field2_index = '1'
```

这个案例已经优化。参见第 10.2.1.3 节，“索引合并优化”。

你也可以通过使用`UNION`来高效解决这个问题，它结合了两个单独的`SELECT`语句的输出。参见第 15.2.18 节，“UNION Clause”。

每个`SELECT`只搜索一个键，并且可以进行优化：

```sql
SELECT field1_index, field2_index
    FROM test_table WHERE field1_index = '1'
UNION
SELECT field1_index, field2_index
    FROM test_table WHERE field2_index = '1';
```
