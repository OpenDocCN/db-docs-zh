# 12.14.2 选择排序规则 ID

> 原文：[`dev.mysql.com/doc/refman/8.0/en/adding-collation-choosing-id.html`](https://dev.mysql.com/doc/refman/8.0/en/adding-collation-choosing-id.html)

每个排序规则必须有一个唯一的 ID。要添加排序规则，必须选择一个当前未使用的 ID 值。MySQL 支持两字节的排序规则 ID。ID 范围从 1024 到 2047 保留给用户定义的排序规则。

你选择的排序规则 ID 出现在以下情境中：

+   信息模式 `COLLATIONS` 表的 `ID` 列。

+   `SHOW COLLATION` 输出的 `Id` 列。

+   `MYSQL_FIELD` C API 数据结构的 `charsetnr` 成员。

+   `mysql_get_character_set_info()` C API 函数返回的 `MY_CHARSET_INFO` 数据结构的 `number` 成员。

要确定当前使用的最大 ID，请执行以下语句：

```sql
mysql> SELECT MAX(ID) FROM INFORMATION_SCHEMA.COLLATIONS;
+---------+
| MAX(ID) |
+---------+
|     247 |
+---------+
```

要显示所有当前使用的 ID 列表，请执行此语句：

```sql
mysql> SELECT ID FROM INFORMATION_SCHEMA.COLLATIONS ORDER BY ID;
+-----+
| ID  |
+-----+
|   1 |
|   2 |
| ... |
|  52 |
|  53 |
|  57 |
|  58 |
| ... |
|  98 |
|  99 |
| 128 |
| 129 |
| ... |
| 247 |
+-----+
```

警告

在升级之前，应保存更改的配置文件。如果在原地升级，该过程将替换修改过的文件。
