> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-definition-data-dumps.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-definition-data-dumps.html)

#### 9.4.5.4 分别转储表定义和内容

`--no-data`选项告诉**mysqldump**不要转储表数据，导致转储文件仅包含创建表的语句。相反，`--no-create-info`选项告诉**mysqldump**从输出中抑制`CREATE`语句，使转储文件仅包含表数据。

例如，要分别为`test`数据库转储表定义和数据，请使用以下命令：

```sql
$> mysqldump --no-data test > dump-defs.sql
$> mysqldump --no-create-info test > dump-data.sql
```

对于仅定义的转储，请添加`--routines`和`--events`选项，以包括存储过程和事件定义：

```sql
$> mysqldump --no-data --routines --events test > dump-defs.sql
```
