> 译文：[`dev.mysql.com/doc/refman/8.0/en/innodb-compression-syntax-warnings.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-syntax-warnings.html)

#### 17.9.1.7 SQL 压缩语法警告和错误

本节描述了在使用表格压缩功能时可能遇到的语法警告和错误，其中包括 file-per-table 表空间和 general tablespaces。

##### 文件-每表表空间的 SQL 压缩语法警告和错误

当`innodb_strict_mode`被启用（默认情况下），在`CREATE TABLE`或`ALTER TABLE`语句中指定`ROW_FORMAT=COMPRESSED`或`KEY_BLOCK_SIZE`会产生以下错误，如果`innodb_file_per_table`被禁用。

```sql
ERROR 1031 (HY000): Table storage engine for 't1' doesn't have this option
```

注意

如果当前配置不允许使用压缩表，表格将不会被创建。

当`innodb_strict_mode`被禁用时，在`CREATE TABLE`或`ALTER TABLE`语句中指定`ROW_FORMAT=COMPRESSED`或`KEY_BLOCK_SIZE`会产生以下警告，如果`innodb_file_per_table`被禁用。

```sql
mysql> SHOW WARNINGS;
+---------+------+---------------------------------------------------------------+
| Level   | Code | Message                                                       |
+---------+------+---------------------------------------------------------------+
| Warning | 1478 | InnoDB: KEY_BLOCK_SIZE requires innodb_file_per_table.        |
| Warning | 1478 | InnoDB: ignoring KEY_BLOCK_SIZE=4\.                            |
| Warning | 1478 | InnoDB: ROW_FORMAT=COMPRESSED requires innodb_file_per_table. |
| Warning | 1478 | InnoDB: assuming ROW_FORMAT=DYNAMIC.                          |
+---------+------+---------------------------------------------------------------+
```

注意

这些消息只是警告，而不是错误，表格是在没有压缩的情况下创建的，就好像没有指定选项一样。

“非严格”行为允许您将`mysqldump`文件导入不支持压缩表的数据库，即使源数据库包含压缩表。在这种情况下，MySQL 会创建表格为`ROW_FORMAT=DYNAMIC`，而不是阻止操作。

要将转储文件导入新数据库，并使表格按照原始数据库中的存在方式重新创建，请确保服务器具有`innodb_file_per_table`配置参数的正确设置。

属性`KEY_BLOCK_SIZE`仅在指定为`COMPRESSED`或省略时才被允许。在任何其他`ROW_FORMAT`中指定`KEY_BLOCK_SIZE`会生成一个警告，您可以通过`SHOW WARNINGS`查看。但是，表格是非压缩的；指定的`KEY_BLOCK_SIZE`会被忽略。

| 等级 | 代码 | 消息 |
| --- | --- | --- |
| 警告 | 1478 | `InnoDB: ignoring KEY_BLOCK_SIZE=*`n`* unless ROW_FORMAT=COMPRESSED.` |

如果您启用了`innodb_strict_mode`，将`KEY_BLOCK_SIZE`与除`COMPRESSED`之外的任何`ROW_FORMAT`组合会产生错误，而不是警告，并且表格不会被创建。

表 17.12，“ROW_FORMAT 和 KEY_BLOCK_SIZE 选项”概述了与`CREATE TABLE`或`ALTER TABLE`一起使用的`ROW_FORMAT`和`KEY_BLOCK_SIZE`选项。

**表 17.12 ROW_FORMAT 和 KEY_BLOCK_SIZE 选项**

| 选项 | 使用说明 | 描述 |
| --- | --- | --- |
| `ROW_FORMAT=​REDUNDANT` | MySQL 5.0.3 之前使用的存储格式 | 比`ROW_FORMAT=COMPACT`效率低；用于向后兼容性 |
| `ROW_FORMAT=​COMPACT` | MySQL 5.0.3 以来的默认存储格式 | 在聚簇索引页中存储长列值的前 768 字节，其余字节存储在溢出页中 |
| `ROW_FORMAT=​DYNAMIC` |  | 如果适合，则在聚簇索引页内存储值；如果不适合，则仅存储一个 20 字节指向溢出页的指针（无前缀） |
| `ROW_FORMAT=​COMPRESSED` |  | 使用 zlib 对表和索引进行压缩 |
| `KEY_BLOCK_​SIZE=*`n`*` |  | 指定压缩页面大小为 1、2、4、8 或 16 千字节；意味着`ROW_FORMAT=COMPRESSED`。对于一般表空间，不允许`KEY_BLOCK_SIZE`值等于`InnoDB`页面大小。 |

表 17.13，“在关闭 InnoDB 严格模式时 CREATE/ALTER TABLE 的警告和错误”总结了在`CREATE TABLE`或`ALTER TABLE`语句中某些配置参数和选项的特定组合导致的错误条件，以及这些选项在`SHOW TABLE STATUS`输出中的显示方式。

当`innodb_strict_mode`为`OFF`时，MySQL 创建或修改表，但会忽略如下设置。您可以在 MySQL 错误日志中看到警告消息。当`innodb_strict_mode`为`ON`时，这些指定的选项组合会生成错误，并且表不会被创建或修改。要查看错误条件的完整描述，请发出`SHOW ERRORS`语句：例如：

```sql
mysql> `CREATE TABLE x (id INT PRIMARY KEY, c INT)` 
-> `ENGINE=INNODB KEY_BLOCK_SIZE=33333;` 
ERROR 1005 (HY000): Can't create table 'test.x' (errno: 1478)

mysql> `SHOW ERRORS;`
+-------+------+-------------------------------------------+
| Level | Code | Message                                   |
+-------+------+-------------------------------------------+
| Error | 1478 | InnoDB: invalid KEY_BLOCK_SIZE=33333\.     |
| Error | 1005 | Can't create table 'test.x' (errno: 1478) |
+-------+------+-------------------------------------------+

```

**表 17.13 在关闭 InnoDB 严格模式时 CREATE/ALTER TABLE 的警告和错误**

| 语法 | 警告或错误条件 | 在`SHOW TABLE STATUS`中显示的`ROW_FORMAT`结果 |
| --- | --- | --- |
| `ROW_FORMAT=REDUNDANT` | 无 | `REDUNDANT` |
| `ROW_FORMAT=COMPACT` | 无 | `COMPACT` |
| 指定了`ROW_FORMAT=COMPRESSED`或`ROW_FORMAT=DYNAMIC`或`KEY_BLOCK_SIZE` | 除非启用了`innodb_file_per_table`，否则对于每个表的文件表空间被忽略。通用表空间支持所有行格式。请参见第 17.6.3.3 节，“General Tablespaces”。 | `文件表空间的默认行格式；通用表空间的指定行格式` |
| 指定了无效的`KEY_BLOCK_SIZE`（不是 1、2、4、8 或 16） | `KEY_BLOCK_SIZE`被忽略 | 指定的行格式，或默认行格式 |
| 指定了`ROW_FORMAT=COMPRESSED`和有效的`KEY_BLOCK_SIZE` | 无；使用指定的`KEY_BLOCK_SIZE` | `COMPRESSED` |
| 指定了`KEY_BLOCK_SIZE`与`REDUNDANT`、`COMPACT`或`DYNAMIC`行格式 | `KEY_BLOCK_SIZE`被忽略 | `REDUNDANT`、`COMPACT`或`DYNAMIC` |
| `ROW_FORMAT`不是`REDUNDANT`、`COMPACT`、`DYNAMIC`或`COMPRESSED`之一 | 如果被 MySQL 解析器识别则被忽略。否则，会发出错误。 | 默认行格式或 N/A |

当`innodb_strict_mode`为`ON`时，MySQL 会拒绝无效的`ROW_FORMAT`或`KEY_BLOCK_SIZE`参数并发出错误。严格模式默认为`ON`。当`innodb_strict_mode`为`OFF`时，MySQL 对被忽略的无效参数发出警告而不是错误。

使用`SHOW TABLE STATUS`无法查看所选的`KEY_BLOCK_SIZE`。语句`SHOW CREATE TABLE`显示`KEY_BLOCK_SIZE`（即使在创建表时被忽略）。MySQL 无法显示表的真实压缩页大小。

##### 通用表空间的 SQL 压缩语法警告和错误

+   如果在创建通用表空间时未定义`FILE_BLOCK_SIZE`，则该表空间不能包含压缩表。如果尝试添加压缩表，则会返回错误，如下例所示：

    ```sql
    mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;

    mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1 ROW_FORMAT=COMPRESSED
           KEY_BLOCK_SIZE=8;
    ERROR 1478 (HY000): InnoDB: Tablespace `ts1` cannot contain a COMPRESSED table
    ```

+   尝试向通用表空间添加具有无效`KEY_BLOCK_SIZE`的表会返回错误，如下例所示：

    ```sql
    mysql> CREATE TABLESPACE `ts2` ADD DATAFILE 'ts2.ibd' FILE_BLOCK_SIZE = 8192 Engine=InnoDB;

    mysql> CREATE TABLE t2 (c1 INT PRIMARY KEY) TABLESPACE ts2 ROW_FORMAT=COMPRESSED
           KEY_BLOCK_SIZE=4;
    ERROR 1478 (HY000): InnoDB: Tablespace `ts2` uses block size 8192 and cannot
    contain a table with physical page size 4096
    ```

    对于通用表空间，表的`KEY_BLOCK_SIZE`必须等于表空间的`FILE_BLOCK_SIZE`除以 1024。例如，如果表空间的`FILE_BLOCK_SIZE`为 8192，则表的`KEY_BLOCK_SIZE`必须为 8。

+   尝试向配置为存储压缩表的通用表空间添加具有未压缩行格式的表会返回错误，如下例所示：

    ```sql
    mysql> CREATE TABLESPACE `ts3` ADD DATAFILE 'ts3.ibd' FILE_BLOCK_SIZE = 8192 Engine=InnoDB;

    mysql> CREATE TABLE t3 (c1 INT PRIMARY KEY) TABLESPACE ts3 ROW_FORMAT=COMPACT;
    ERROR 1478 (HY000): InnoDB: Tablespace `ts3` uses block size 8192 and cannot
    contain a table with physical page size 16384
    ```

`innodb_strict_mode`不适用于通用表空间。通用表空间的表空间管理规则严格执行，与`innodb_strict_mode`无关。有关更多信息，请参见第 15.1.21 节，“CREATE TABLESPACE Statement”。

有关在通用表空间中使用压缩表的更多信息，请参阅第 17.6.3.3 节，“通用表空间”。
