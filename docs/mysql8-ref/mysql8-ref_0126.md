# 5.3.4 从表中检索信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/retrieving-data.html`](https://dev.mysql.com/doc/refman/8.0/en/retrieving-data.html)

5.3.4.1 选择所有数据

5.3.4.2 选择特定行

5.3.4.3 选择特定列

5.3.4.4 排序行

5.3.4.5 日期计算

5.3.4.6 处理 NULL 值

5.3.4.7 模式匹配

5.3.4.8 计算行数

5.3.4.9 使用多个表

`SELECT` 语句用于从表中提取信息。语句的一般形式为：

```sql
SELECT *what_to_select*
FROM *which_table*
WHERE *conditions_to_satisfy*;
```

*`what_to_select`* 表示您想要查看的内容。这可以是列的列表，或者使用 `*` 表示“所有列”。 *`which_table`* 表示您要从中检索数据的表。`WHERE` 子句是可选的。如果存在，*`conditions_to_satisfy`* 指定行必须满足的一个或多个条件才能符合检索条件。
