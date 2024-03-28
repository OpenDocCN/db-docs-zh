# 28.3.7 INFORMATION_SCHEMA COLLATION_CHARACTER_SET_APPLICABILITY 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-collation-character-set-applicability-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-collation-character-set-applicability-table.html)

`COLLATION_CHARACTER_SET_APPLICABILITY` 表示适用于哪种排序的字符集。

`COLLATION_CHARACTER_SET_APPLICABILITY` 表包含以下列：

+   `COLLATION_NAME`

    排序名称。

+   `CHARACTER_SET_NAME`

    与排序关联的字符集的名称。

#### 注意事项

`COLLATION_CHARACTER_SET_APPLICABILITY` 的列等同于 `SHOW COLLATION` 语句显示的前两列。
