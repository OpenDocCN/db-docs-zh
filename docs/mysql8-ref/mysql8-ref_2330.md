> 原文：[`dev.mysql.com/doc/refman/8.0/en/no-matching-rows.html`](https://dev.mysql.com/doc/refman/8.0/en/no-matching-rows.html)

#### B.3.4.7 解决没有匹配行的问题

如果您有一个使用许多表但不返回任何行的复杂查询，您应该使用以下过程找出问题所在：

1.  使用`EXPLAIN`测试查询，以检查是否可以找到明显错误。参见第 15.8.2 节，“EXPLAIN Statement”。

1.  仅选择在`WHERE`子句中使用的列。

1.  从查询中逐个删除一个表，直到返回一些行。如果表很大，最好在查询中使用`LIMIT 10`。

1.  针对应该与上次从查询中删除的表匹配的列发出一个`SELECT`。

1.  如果您正在比较带有小数的数字的`FLOAT` - FLOAT, DOUBLE")或`DOUBLE` - FLOAT, DOUBLE")列，您不能使用相等（`=`）比较。这个问题在大多数计算机语言中很常见，因为并非所有浮点值都可以精确存储。在某些情况下，将`FLOAT` - FLOAT, DOUBLE")更改为`DOUBLE` - FLOAT, DOUBLE")可以解决这个问题。参见第 B.3.4.8 节，“浮点值的问题”。

1.  如果您仍然无法找出问题所在，请创建一个最小的测试，可以通过`mysql test < query.sql`运行，显示您的问题。您可以通过使用**mysqldump --quick db_name *`tbl_name_1`* ... *`tbl_name_n`* > query.sql**转储表来创建一个测试文件。在编辑器中打开文件，删除一些插入行（如果有多余的行来展示问题），并在文件末尾添加您的`SELECT`语句。

    通过执行以下命令验证测试文件是否展示了问题：

    ```sql
    $> mysqladmin create test2
    $> mysql test2 < query.sql
    ```

    将测试文件附加到一个 bug 报告中，您可以按照第 1.5 节，“如何报告错误或问题”中的说明进行报告。
