> 原文：[`dev.mysql.com/doc/refman/8.0/en/static-format.html`](https://dev.mysql.com/doc/refman/8.0/en/static-format.html)

#### 18.2.3.1 静态（固定长度）表特性

静态格式是`MyISAM`表的默认格式。当表中不包含可变长度列（`VARCHAR`、`VARBINARY`、`BLOB`或`TEXT`）时使用。每行使用固定字节数存储。

在三种`MyISAM`存储格式中，静态格式是最简单和最安全（最不容易损坏）的。由于在磁盘上很容易找到数据文件中的行，它也是磁盘上最快的格式：根据索引中的行号查找行时，将行号乘以行长度以计算行位置。此外，在扫描表时，每次磁盘读取操作都可以很容易地读取恒定数量的行。

如果在 MySQL 服务器写入固定格式的`MyISAM`文件时计算机崩溃，安全性得到证明。在这种情况下，**myisamchk**可以轻松确定每行的起始和结束位置，因此通常可以回收除部分写入的行之外的所有行。`MyISAM`表索引始终可以根据数据行重建。

注意

仅适用于没有`BLOB`或`TEXT`列的表的固定长度行格式。创建具有此类列的表并使用显式的`ROW_FORMAT`子句不会引发错误或警告；格式规范将被忽略。

静态格式表具有以下特点：

+   `CHAR`和`VARCHAR`列会被填充到指定的列宽，尽管列类型不会改变。`BINARY`和`VARBINARY`列会用`0x00`字节填充到列宽。

+   `NULL`列需要额外的空间来记录它们的值是否为`NULL`。每个`NULL`列额外占用一位，向最近的字节取整。

+   非常快速。

+   易于缓存。

+   在崩溃后易于重建，因为行位于固定位置。

+   除非删除大量行并希望将空闲磁盘空间返回给操作系统，否则不需要重新组织。要执行此操作，请使用`OPTIMIZE TABLE`或**myisamchk -r**。

+   通常需要比动态格式表更多的磁盘空间。

+   静态大小行的预期行长度（以字节为单位）通过以下表达式计算：

    ```sql
    row length = 1
                 + (*sum of column lengths*)
                 + (*number of NULL columns* + *delete_flag* + 7)/8
                 + (*number of variable-length columns*)
    ```

    *`delete_flag`* 在静态行格式的表中为 1。静态表在行记录中使用一个位来表示行是否已被删除。对于动态表，*`delete_flag`* 为 0，因为标志位存储在动态行头中。
