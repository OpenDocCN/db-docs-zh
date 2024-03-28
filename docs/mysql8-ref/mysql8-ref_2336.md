# MySQL 中的已知问题 B.3.7。

> 原文：[`dev.mysql.com/doc/refman/8.0/en/known-issues.html`](https://dev.mysql.com/doc/refman/8.0/en/known-issues.html)

本节列出了 MySQL 最新版本中已知的问题。

有关特定平台问题的信息，请参阅第 2.1 节“一般安装指南”和第 7.9 节“调试 MySQL”中的安装和调试说明。

已知以下问题：

+   `IN`的子查询优化不如`=`有效。

+   即使使用`lower_case_table_names=2`（使 MySQL 记住数据库和表名的大小写），MySQL 也不会记住数据库名称在函数`DATABASE()`或各种日志中使用的大小写（在不区分大小写的系统上）。

+   在复制中删除`FOREIGN KEY`约束不起作用，因为约束在副本上可能有另一个名称。

+   `REPLACE`（以及带有`REPLACE`选项的`LOAD DATA``](aggregate-functions.html#function_group-concat)内部不起作用。

+   当将大整数值（介于 2⁶³和 2⁶⁴−1 之间）插入十进制或字符串列时，它将作为负值插入，因为该数字在有符号整数上下文中进行评估。

+   使用基于语句的二进制日志记录，源服务器将执行的查询写入二进制日志。这是一种非常快速、紧凑和高效的日志记录方法，在大多数情况下都能完美运行。但是，如果查询设计为数据修改是非确定性的（通常不建议的做法，即使在复制之外），则源和副本上的数据可能会变得不同。

    例如：

    +   `CREATE TABLE ... SELECT`或`INSERT ... SELECT`语句将零值或`NULL`值插入`AUTO_INCREMENT`列。

    +   `DELETE`如果您正在从具有`ON DELETE CASCADE`属性的外键的表中删除行。

    +   `REPLACE ... SELECT`，`INSERT IGNORE ... SELECT`如果插入的数据中有重复的键值。

    **仅当前面的查询没有保证确定性顺序的`ORDER BY`子句时**。

    例如，对于没有 `ORDER BY` 的 `INSERT ... SELECT`，`SELECT` 可能以不同的顺序返回行（这导致行具有不同的排名，因此在 `AUTO_INCREMENT` 列中获得不同的数字），这取决于源和复制品上优化器所做的选择。

    只有在源和复制品上查询被优化不同的情况下才会有不同的情况：

    +   表在源上使用不同的存储引擎存储，而在复制品上使用不同的存储引擎。 （在源和复制品上可以使用不同的存储引擎。例如，如果复制品的可用磁盘空间较少，可以在源上使用 `InnoDB`，但在复制品上使用 `MyISAM`。）

    +   MySQL 缓冲区大小（`key_buffer_size` 等）在源和复制品上是不同的。

    +   源和复制品运行不同的 MySQL 版本，并且这些版本之间的优化器代码不同。

    这个问题也可能影响使用 **mysqlbinlog|mysql** 进行数据库恢复。

    避免这个问题的最简单方法是为上述不确定性查询添加一个 `ORDER BY` 子句，以确保行始终以相同的顺序存储或修改。使用基于行或混合日志格式也可以避免这个问题。

+   如果您没有使用启动选项指定文件名，则日志文件名基于服务器主机名。如果您将主机名更改为其他内容，要保留相同的日志文件名，必须显式使用选项，如 `--log-bin=*`old_host_name`*-bin`。请参阅 Section 7.1.7, “Server Command Options”。或者，将旧文件重命名以反映您的主机名更改。如果这些是二进制日志，则必须编辑二进制日志索引文件并在那里修复二进制日志文件名。（对于复制品上的中继日志也是如此。）

+   **mysqlbinlog** 不会删除 `LOAD DATA` 语句留下的临时文件。请参阅 Section 6.6.9, “mysqlbinlog — Utility for Processing Binary Log Files”。

+   `RENAME` 不能用于 `TEMPORARY` 表或用于 `MERGE` 表的表。

+   使用 `SET CHARACTER SET` 时，不能在数据库、表和列名中使用翻译字符。

+   在 MySQL 8.0.17 之前，你不能在 `LIKE ... ESCAPE` 中使用 `_` 或 `%` 与 `ESCAPE`。

+   服务器在比较数据值时只使用前`max_sort_length`字节。这意味着如果值仅在前`max_sort_length`字节之后有差异，则不能可靠地在`GROUP BY`、`ORDER BY`或`DISTINCT`中使用这些值。为了解决这个问题，增加变量值。`max_sort_length`的默认值为 1024，可以在服务器启动时或运行时更改。

+   数值计算使用`BIGINT`或`DOUBLE`（通常都是 64 位长）。你得到的精度取决于函数。一般规则是，位函数使用`BIGINT`精度，`IF()`和`ELT()`使用`BIGINT`或`DOUBLE`精度，其余使用`DOUBLE`精度。除了位字段之外，应尽量避免使用无符号长长整型值，如果它们解析为大于 63 位（9223372036854775807）的值。

+   在一个表中可以有多达 255 个`ENUM`和`SET`列。

+   在`MIN()`、`MAX()`和其他聚合函数中，MySQL 当前通过字符串值而不是字符串在集合中的相对位置来比较`ENUM`和`SET`列。

+   在一个`UPDATE`语句中，列从左到右更新。如果引用已更新的列，你将得到更新后的值而不是原始值。例如，以下语句将`KEY`增加`2`，而**不是**`1`：

    ```sql
    mysql> UPDATE *tbl_name* SET KEY=KEY+1,KEY=KEY+1;
    ```

+   你可以在同一查询中引用多个临时表，但不能多次引用任何给定的临时表。例如，以下操作不起作用：

    ```sql
    mysql> SELECT * FROM temp_table, temp_table AS t2;
    ERROR 1137: Can't reopen table: 'temp_table'
    ```

+   当你在连接中使用“隐藏”列时，优化器可能会以不同的方式处理`DISTINCT`，而在普通查询中，隐藏列不参与`DISTINCT`比较。

    一个例子是：

    ```sql
    SELECT DISTINCT mp3id FROM band_downloads
           WHERE userid = 9 ORDER BY id DESC;
    ```

    和

    ```sql
    SELECT DISTINCT band_downloads.mp3id
           FROM band_downloads,band_mp3
           WHERE band_downloads.userid = 9
           AND band_mp3.id = band_downloads.mp3id
           ORDER BY band_downloads.id DESC;
    ```

    在第二种情况下，您可能会在结果集中得到两行相同的行（因为隐藏的`id`列中的值可能不同）。

    请注意，这仅适用于结果中没有`ORDER BY`列的查询。

+   如果在返回空集的查询上执行一个`PROCEDURE`，在某些情况下，`PROCEDURE`不会转换列。

+   创建一个`MERGE`类型的表不会检查底层表是否是兼容类型。

+   如果您使用`ALTER TABLE`向用于`MERGE`表的表添加一个`UNIQUE`索引，然后在`MERGE`表上添加一个普通索引，如果表中有旧的非`UNIQUE`键，则表的键顺序会不同。这是因为`ALTER TABLE`将`UNIQUE`索引放在普通索引之前，以便尽早检测到重复键。
