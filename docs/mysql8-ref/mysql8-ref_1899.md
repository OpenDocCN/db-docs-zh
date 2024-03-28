# 28.3.4 The INFORMATION_SCHEMA CHARACTER_SETS Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-character-sets-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-character-sets-table.html)

`CHARACTER_SETS` 表提供了关于可用字符集的信息。

`CHARACTER_SETS` 表包含以下列：

+   `CHARACTER_SET_NAME`

    字符集名称。

+   `DEFAULT_COLLATE_NAME`

    字符集的默认排序规则。

+   `DESCRIPTION`

    字符集的描述。

+   `MAXLEN`

    存储一个字符所需的最大字节数。

#### 注意

字符集信息也可以通过 `SHOW CHARACTER SET` 语句获取。参见 Section 15.7.7.3, “SHOW CHARACTER SET Statement”。以下语句是等效的：

```sql
SELECT * FROM INFORMATION_SCHEMA.CHARACTER_SETS
  [WHERE CHARACTER_SET_NAME LIKE '*wild*']

SHOW CHARACTER SET
  [LIKE '*wild*']
```
