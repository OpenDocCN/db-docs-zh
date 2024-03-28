# 28.3.6 INFORMATION_SCHEMA COLLATIONS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-collations-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-collations-table.html)

`COLLATIONS` 表提供了每个字符集的排序规则信息。

`COLLATIONS` 表包含以下列：

+   `COLLATION_NAME`

    排序规则名称。

+   `CHARACTER_SET_NAME`

    与排序规则相关联的字符集的名称。

+   `ID`

    排序规则 ID。

+   `IS_DEFAULT`

    排序规则是否是其字符集的默认规则。

+   `IS_COMPILED`

    字符集是否编译到服务器中。

+   `SORTLEN`

    这与在字符集中表达的字符串所需的排序所需的内存量有关。

+   `PAD_ATTRIBUTE`

    排序规则填充属性，可以是 `NO PAD` 或 `PAD SPACE`。此属性影响在字符串比较中是否尾随空格有意义；请参阅比较中的尾随空格处理。

#### 注意事项

排序规则信息也可以从 `SHOW COLLATION` 语句中获取。请参阅 Section 15.7.7.4, “SHOW COLLATION 语句”。以下语句是等效的：

```sql
SELECT COLLATION_NAME FROM INFORMATION_SCHEMA.COLLATIONS
  [WHERE COLLATION_NAME LIKE '*wild*']

SHOW COLLATION
  [LIKE '*wild*']
```
