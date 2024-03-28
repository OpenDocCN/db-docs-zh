# 12.3.5 列字符集和排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-column.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-column.html)

每个“字符”列（即`CHAR`、`VARCHAR`类型的列，`TEXT`类型，或任何同义词）都有一个列字符集和一个列排序规则。用于`CREATE TABLE`和`ALTER TABLE`的列定义语法具有可选子句，用于指定列字符集和排序规则：

```sql
*col_name* {CHAR | VARCHAR | TEXT} (*col_length*)
    [CHARACTER SET *charset_name*]
    [COLLATE *collation_name*]
```

这些子句也可用于`ENUM`和`SET`列：

```sql
*col_name* {ENUM | SET} (*val_list*)
    [CHARACTER SET *charset_name*]
    [COLLATE *collation_name*]
```

例如：

```sql
CREATE TABLE t1
(
    col1 VARCHAR(5)
      CHARACTER SET latin1
      COLLATE latin1_german1_ci
);

ALTER TABLE t1 MODIFY
    col1 VARCHAR(5)
      CHARACTER SET latin1
      COLLATE latin1_swedish_ci;
```

MySQL 选择列的字符集和排序规则的方式如下：

+   如果同时指定了`CHARACTER SET *`charset_name`*`和`COLLATE *`collation_name`*`，则使用字符集*`charset_name`*和排序规则*`collation_name`*。

    ```sql
    CREATE TABLE t1
    (
        col1 CHAR(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
    ) CHARACTER SET latin1 COLLATE latin1_bin;
    ```

    该列已指定字符集和排序规则，因此使用它们。该列的字符集为`utf8mb4`，排序规则为`utf8mb4_unicode_ci`。

+   如果指定了`CHARACTER SET *`charset_name`*`但未指定`COLLATE`，则使用字符集*`charset_name`*及其默认排序规则。

    ```sql
    CREATE TABLE t1
    (
        col1 CHAR(10) CHARACTER SET utf8mb4
    ) CHARACTER SET latin1 COLLATE latin1_bin;
    ```

    对于该列，指定了字符集，但未指定排序规则。该列的字符集为`utf8mb4`，默认排序规则为`utf8mb4_0900_ai_ci`。要查看每个字符集的默认排序规则，请使用`SHOW CHARACTER SET`语句或查询`INFORMATION_SCHEMA` `CHARACTER_SETS`表。

+   如果指定了`COLLATE *`collation_name`*`但未指定`CHARACTER SET`，则使用与*`collation_name`*相关联的字符集和排序规则*`collation_name`*。

    ```sql
    CREATE TABLE t1
    (
        col1 CHAR(10) COLLATE utf8mb4_polish_ci
    ) CHARACTER SET latin1 COLLATE latin1_bin;
    ```

    对于该列，指定了排序规则，但未指定字符集。该列的排序规则为`utf8mb4_polish_ci`，字符集为与排序规则相关联的字符集，即`utf8mb4`。

+   否则（既未指定`CHARACTER SET`也未指定`COLLATE`），则使用表的字符集和排序规则。

    ```sql
    CREATE TABLE t1
    (
        col1 CHAR(10)
    ) CHARACTER SET latin1 COLLATE latin1_bin;
    ```

    对于该列，既未指定字符集也未指定排序规则，因此使用表的默认设置。该列的字符集为`latin1`，排序规则为`latin1_bin`。

`CHARACTER SET`和`COLLATE`子句是标准 SQL。

如果使用`ALTER TABLE`将列从一个字符集转换为另一个字符集，MySQL 会尝试映射数据值，但如果字符集不兼容，则可能会丢失数据。
