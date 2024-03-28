> 原文：[`dev.mysql.com/doc/refman/8.0/en/ansi-diff-select-into-table.html`](https://dev.mysql.com/doc/refman/8.0/en/ansi-diff-select-into-table.html)

#### 1.6.2.1 SELECT INTO TABLE 差异

MySQL Server 不支持`SELECT ... INTO TABLE` Sybase SQL 扩展。相反，MySQL Server 支持标准 SQL 语法`INSERT INTO ... SELECT`，基本上是相同的事情。参见 Section 15.2.7.1, “INSERT ... SELECT Statement”。例如：

```sql
INSERT INTO tbl_temp2 (fld_id)
    SELECT tbl_temp1.fld_order_id
    FROM tbl_temp1 WHERE tbl_temp1.fld_order_id > 100;
```

或者，您可以使用`SELECT ... INTO OUTFILE`或`CREATE TABLE ... SELECT`。

您可以使用带有用户定义变量的`SELECT ... INTO`。相同的语法也可以在使用游标和本地变量的存储过程中使用。参见 Section 15.2.13.1, “SELECT ... INTO Statement”。
