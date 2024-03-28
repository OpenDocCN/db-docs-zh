> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-collation.html`](https://dev.mysql.com/doc/refman/8.0/en/show-collation.html)

#### 15.7.7.4 SHOW COLLATION Statement

```sql
SHOW COLLATION
    [LIKE '*pattern*' | WHERE *expr*]
```

此语句列出了服务器支持的排序规则。默认情况下，`SHOW COLLATION` 的输出包括所有可用的排序规则。如果存在 `LIKE` 子句，则指示要匹配的排序规则名称。可以使用 `WHERE` 子句来选择使用更一般条件的行，如 第 28.8 节，“SHOW 语句的扩展” 中讨论的那样。例如：

```sql
mysql> SHOW COLLATION WHERE Charset = 'latin1';
+-------------------+---------+----+---------+----------+---------+
| Collation         | Charset | Id | Default | Compiled | Sortlen |
+-------------------+---------+----+---------+----------+---------+
| latin1_german1_ci | latin1  |  5 |         | Yes      |       1 |
| latin1_swedish_ci | latin1  |  8 | Yes     | Yes      |       1 |
| latin1_danish_ci  | latin1  | 15 |         | Yes      |       1 |
| latin1_german2_ci | latin1  | 31 |         | Yes      |       2 |
| latin1_bin        | latin1  | 47 |         | Yes      |       1 |
| latin1_general_ci | latin1  | 48 |         | Yes      |       1 |
| latin1_general_cs | latin1  | 49 |         | Yes      |       1 |
| latin1_spanish_ci | latin1  | 94 |         | Yes      |       1 |
+-------------------+---------+----+---------+----------+---------+
```

`SHOW COLLATION` 的输出包括以下列：

+   `Collation`

    排序规则名称。

+   `Charset`

    与排序规则关联的字符集的名称。

+   `Id`

    排序规则 ID。

+   `Default`

    排序规则是否是其字符集的默认值。

+   `Compiled`

    字符集是否编译到服务器中。

+   `Sortlen`

    这与在字符集中表达的字符串所需的排序所需的内存量有关。

要查看每个字符集的默认排序规则，请使用以下语句。`Default`是一个保留字，因此要将其用作标识符，必须将其引用为：

```sql
mysql> SHOW COLLATION WHERE `Default` = 'Yes';
+---------------------+----------+----+---------+----------+---------+
| Collation           | Charset  | Id | Default | Compiled | Sortlen |
+---------------------+----------+----+---------+----------+---------+
| big5_chinese_ci     | big5     |  1 | Yes     | Yes      |       1 |
| dec8_swedish_ci     | dec8     |  3 | Yes     | Yes      |       1 |
| cp850_general_ci    | cp850    |  4 | Yes     | Yes      |       1 |
| hp8_english_ci      | hp8      |  6 | Yes     | Yes      |       1 |
| koi8r_general_ci    | koi8r    |  7 | Yes     | Yes      |       1 |
| latin1_swedish_ci   | latin1   |  8 | Yes     | Yes      |       1 |
...
```

排序规则信息也可以从 `INFORMATION_SCHEMA` 的 `COLLATIONS` 表中获取。请参见 第 28.3.6 节，“INFORMATION_SCHEMA COLLATIONS 表”。
