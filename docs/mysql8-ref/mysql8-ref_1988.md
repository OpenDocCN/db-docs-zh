# 28.8 `SHOW` 语句的扩展

> 原文：[`dev.mysql.com/doc/refman/8.0/en/extended-show.html`](https://dev.mysql.com/doc/refman/8.0/en/extended-show.html)

一些扩展到 `SHOW` 语句伴随着 `INFORMATION_SCHEMA` 的实现：

+   `SHOW` 可以用于获取关于 `INFORMATION_SCHEMA` 本身结构的信息。

+   几个 `SHOW` 语句接受一个提供更灵活性的 `WHERE` 子句，用于指定要显示哪些行。

`INFORMATION_SCHEMA` 是一个信息数据库，因此其名称包含在 `SHOW DATABASES` 的输出中。类似地，`SHOW TABLES` 可以与 `INFORMATION_SCHEMA` 一起使用以获取其表的列表：

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA;
+---------------------------------------+
| Tables_in_INFORMATION_SCHEMA          |
+---------------------------------------+
| CHARACTER_SETS                        |
| COLLATIONS                            |
| COLLATION_CHARACTER_SET_APPLICABILITY |
| COLUMNS                               |
| COLUMN_PRIVILEGES                     |
| ENGINES                               |
| EVENTS                                |
| FILES                                 |
| KEY_COLUMN_USAGE                      |
| PARTITIONS                            |
| PLUGINS                               |
| PROCESSLIST                           |
| REFERENTIAL_CONSTRAINTS               |
| ROUTINES                              |
| SCHEMATA                              |
| SCHEMA_PRIVILEGES                     |
| STATISTICS                            |
| TABLES                                |
| TABLE_CONSTRAINTS                     |
| TABLE_PRIVILEGES                      |
| TRIGGERS                              |
| USER_PRIVILEGES                       |
| VIEWS                                 |
+---------------------------------------+
```

`SHOW COLUMNS` 和 `DESCRIBE` 可以显示关于各个 `INFORMATION_SCHEMA` 表中列的信息。

`SHOW` 语句接受一个 `LIKE` 子句来限制显示的行，也允许使用 `WHERE` 子句来指定所选行必须满足的更一般条件：

```sql
SHOW CHARACTER SET
SHOW COLLATION
SHOW COLUMNS
SHOW DATABASES
SHOW FUNCTION STATUS
SHOW INDEX
SHOW OPEN TABLES
SHOW PROCEDURE STATUS
SHOW STATUS
SHOW TABLE STATUS
SHOW TABLES
SHOW TRIGGERS
SHOW VARIABLES
```

如果存在 `WHERE` 子句，则将其针对 `SHOW` 语句显示的列名进行评估。例如，`SHOW CHARACTER SET` 语句产生以下输出列：

```sql
mysql> SHOW CHARACTER SET;
+----------+-----------------------------+---------------------+--------+
| Charset  | Description                 | Default collation   | Maxlen |
+----------+-----------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese    | big5_chinese_ci     |      2 |
| dec8     | DEC West European           | dec8_swedish_ci     |      1 |
| cp850    | DOS West European           | cp850_general_ci    |      1 |
| hp8      | HP West European            | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian       | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European        | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European | latin2_general_ci   |      1 |
...
```

要在 `SHOW CHARACTER SET` 中使用 `WHERE` 子句，您需要引用那些列名。例如，以下语句显示了默认排序包含字符串 `'japanese'` 的字符集的信息：

```sql
mysql> SHOW CHARACTER SET WHERE `Default collation` LIKE '%japanese%';
+---------+---------------------------+---------------------+--------+
| Charset | Description               | Default collation   | Maxlen |
+---------+---------------------------+---------------------+--------+
| ujis    | EUC-JP Japanese           | ujis_japanese_ci    |      3 |
| sjis    | Shift-JIS Japanese        | sjis_japanese_ci    |      2 |
| cp932   | SJIS for Windows Japanese | cp932_japanese_ci   |      2 |
| eucjpms | UJIS for Windows Japanese | eucjpms_japanese_ci |      3 |
+---------+---------------------------+---------------------+--------+
```

此语句显示了多字节字符集：

```sql
mysql> SHOW CHARACTER SET WHERE Maxlen > 1;
+---------+---------------------------------+---------------------+--------+
| Charset | Description                     | Default collation   | Maxlen |
+---------+---------------------------------+---------------------+--------+
| big5    | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
| cp932   | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
| eucjpms | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
| euckr   | EUC-KR Korean                   | euckr_korean_ci     |      2 |
| gb18030 | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
| gb2312  | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
| gbk     | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
| sjis    | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
| ucs2    | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
| ujis    | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
| utf16   | UTF-16 Unicode                  | utf16_general_ci    |      4 |
| utf16le | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
| utf32   | UTF-32 Unicode                  | utf32_general_ci    |      4 |
| utf8mb3 | UTF-8 Unicode                   | utf8mb3_general_ci  |      3 |
| utf8mb4 | UTF-8 Unicode                   | utf8mb4_0900_ai_ci  |      4 |
+---------+---------------------------------+---------------------+--------+
```
