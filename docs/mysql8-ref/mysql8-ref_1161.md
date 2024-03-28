> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-table-import.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-table-import.html)

#### 17.6.1.3 导入 InnoDB 表

本节描述如何使用*可传输表空间*功能导入表，该功能允许导入位于文件-每表表空间中的表、分区表或单个表分区。有许多原因您可能想要导入表：

+   在非生产 MySQL 服务器实例上运行报告，以避免对生产服务器造成额外负载。

+   将数据复制到新的副本服务器。

+   从备份的表空间文件中恢复表。

+   作为比导入转储文件更快的移动数据的方法，后者需要重新插入数据和重建索引。

+   将数据移动到一个存储介质更适合您存储需求的服务器上。例如，您可以将繁忙的表移动到 SSD 设备，或将大表移动到高容量的 HDD 设备。

*可传输表空间*功能在本节中的以下主题中进行了描述：

+   先决条件

+   导入表

+   导入分区表

+   导入表分区

+   限制

+   使用说明

+   内部结构

##### 先决条件

+   `innodb_file_per_table`变量必须启用，默认情况下已启用。

+   表空间的页面大小必须与目标 MySQL 服务器实例的页面大小匹配。`InnoDB`页面大小由`innodb_page_size`变量定义，在初始化 MySQL 服务器实例时配置。

+   如果表具有外键关系，在执行`DISCARD TABLESPACE`之前必须禁用`foreign_key_checks`。此外，应在同一逻辑时间点导出所有相关的外键表，因为`ALTER TABLE ... IMPORT TABLESPACE`不会对导入的数据强制执行外键约束。为此，请停止更新相关表，提交所有事务，在表上获取共享锁，并执行导出操作。

+   当从另一个 MySQL 服务器实例导入表时，两个 MySQL 服务器实例必须具有通用可用性（GA）状态，并且必须是相同版本。否则，表必须在导入的 MySQL 服务器实例中创建。

+   如果表是通过在`CREATE TABLE`语句中指定`DATA DIRECTORY`子句在外部目录中创建的，则在目标实例上替换的表必须使用相同的`DATA DIRECTORY`子句进行定义。如果子句不匹配，则会报告模式不匹配错误。要确定源表是否使用`DATA DIRECTORY`子句定义，请使用`SHOW CREATE TABLE`查看表定义。有关使用`DATA DIRECTORY`子句的信息，请参见第 17.6.1.2 节，“外部创建表”。

+   如果表定义中未明确定义`ROW_FORMAT`选项或使用了`ROW_FORMAT=DEFAULT`，则源实例和目标实例上的`innodb_default_row_format`设置必须相同。否则，在尝试导入操作时会报告模式不匹配错误。使用`SHOW CREATE TABLE`检查表定义。使用`SHOW VARIABLES`检查`innodb_default_row_format`设置。有关相关信息，请参见定义表的行格式。

##### 导入表

此示例演示了如何导入一个位于文件表空间中的常规非分区表。

1.  在目标实例上，创建一个与您打算导入的表具有相同定义的表。（您可以使用`SHOW CREATE TABLE`语法获取表定义。）如果表定义不匹配，在尝试导入操作时会报告模式不匹配错误。

    ```sql
    mysql> USE test;
    mysql> CREATE TABLE t1 (c1 INT) ENGINE=INNODB;
    ```

1.  在目标实例上，丢弃刚刚创建的表的表空间。（在导入之前，您必须丢弃接收表的表空间。）

    ```sql
    mysql> ALTER TABLE t1 DISCARD TABLESPACE;
    ```

1.  在源实例上，运行`FLUSH TABLES ... FOR EXPORT`以使您打算导入的表静止。当表被静止时，只允许对表进行只读事务。

    ```sql
    mysql> USE test;
    mysql> FLUSH TABLES t1 FOR EXPORT;
    ```

    `FLUSH TABLES ... FOR EXPORT`确保对命名表的更改刷新到磁盘，以便在服务器运行时可以进行二进制表复制。运行`FLUSH TABLES ... FOR EXPORT`时，`InnoDB`在表的模式目录中生成一个`.cfg`元数据文件。`.cfg`文件包含在导入操作期间用于模式验证的元数据。

    注意

    执行`FLUSH TABLES ... FOR EXPORT`的连接在操作运行时必须保持打开状态；否则，随着连接关闭，`.cfg`文件将被删除，因为锁在连接关闭时被释放。

1.  从源实例复制`.ibd`文件和`.cfg`元数据文件到目标实例。例如：

    ```sql
    $> scp */path/to/datadir*/test/t1.{ibd,cfg} destination-server:*/path/to/datadir*/test
    ```

    在释放共享锁之前，必须复制`.ibd`文件和`.cfg`文件，如下一步骤所述。

    注意

    如果您从加密表空间导入表，`InnoDB`会生成一个`.cfp`文件，除了一个`.cfg`元数据文件。`.cfp`文件必须与`.cfg`文件一起复制到目标实例。`.cfp`文件包含一个传输密钥和一个加密表空间密钥。在导入时，`InnoDB`使用传输密钥解密表空间密钥。有关相关信息，请参见第 17.13 节，“InnoDB 数据静止加密”。

1.  在源实例上，使用`UNLOCK TABLES`释放`FLUSH TABLES ... FOR EXPORT`语句获取的锁：

    ```sql
    mysql> USE test;
    mysql> UNLOCK TABLES;
    ```

    `UNLOCK TABLES`操作也会删除`.cfg`文件。

1.  在目标实例上，导入表空间：

    ```sql
    mysql> USE test;
    mysql> ALTER TABLE t1 IMPORT TABLESPACE;
    ```

##### 导入分区表

本示例演示了如何导入一个分区表，其中每个表分区都驻留在一个文件表表空间中。

1.  在目标实例上，创建一个与要导入的分区表相同定义的分区表。（您可以使用`SHOW CREATE TABLE`语法获取表定义。）如果表定义不匹配，则在尝试导入操作时会报告模式不匹配错误。

    ```sql
    mysql> USE test;
    mysql> CREATE TABLE t1 (i int) ENGINE = InnoDB PARTITION BY KEY (i) PARTITIONS 3;
    ```

    在`/*`datadir`*/test`目录中，每个分区都有一个`.ibd`文件的表空间。

    ```sql
    mysql> \! ls */path/to/datadir*/test/
    t1#p#p0.ibd  t1#p#p1.ibd  t1#p#p2.ibd
    ```

1.  在目标实例上，丢弃分区表的表空间。（在导入操作之前，必须丢弃接收表的表空间。）

    ```sql
    mysql> ALTER TABLE t1 DISCARD TABLESPACE;
    ```

    分区表的三个表空间`.ibd`文件将从`/*`datadir`*/test`目录中丢弃。

1.  在源实例上，运行`FLUSH TABLES ... FOR EXPORT`来使您打算导入的分区表静止。当表被静止时，只允许对表进行只读事务。

    ```sql
    mysql> USE test;
    mysql> FLUSH TABLES t1 FOR EXPORT;
    ```

    `FLUSH TABLES ... FOR EXPORT`确保将对命名表的更改刷新到磁盘，以便在服务器运行时可以进行二进制表复制。当运行`FLUSH TABLES ... FOR EXPORT`时，`InnoDB`为表的每个表空间文件在表的模式目录中生成`.cfg`元数据文件。

    ```sql
    mysql> \! ls */path/to/datadir*/test/
    t1#p#p0.ibd  t1#p#p1.ibd  t1#p#p2.ibd
    t1#p#p0.cfg  t1#p#p1.cfg  t1#p#p2.cfg
    ```

    `.cfg`文件包含在导入表空间时用于模式验证的元数据。`FLUSH TABLES ... FOR EXPORT`只能在表上运行，而不能在单个表分区上运行。

1.  将`.ibd`和`.cfg`文件从源实例模式目录复制到目标实例模式目录。例如：

    ```sql
    $>scp */path/to/datadir*/test/t1*.{ibd,cfg} destination-server:*/path/to/datadir*/test
    ```

    在释放共享锁之前，必须复制`.ibd`和`.cfg`文件，如下一步所述。

    注意

    如果从加密表空间导入表，则`InnoDB`会生成一个`.cfp`文件，除了一个`.cfg`元数据文件。`.cfp`文件必须与`.cfg`文件一起复制到目标实例。`.cfp`文件包含传输密钥和加密表空间密钥。在导入时，`InnoDB`使用传输密钥解密表空间密钥。有关相关信息，请参见第 17.13 节，“InnoDB 数据静止加密”。

1.  在源实例上，使用`UNLOCK TABLES`释放由`FLUSH TABLES ... FOR EXPORT`获取的锁：

    ```sql
    mysql> USE test;
    mysql> UNLOCK TABLES;
    ```

1.  在目标实例上，导入分区表的表空间：

    ```sql
    mysql> USE test;
    mysql> ALTER TABLE t1 IMPORT TABLESPACE;
    ```

##### 导入表分区

本示例演示了如何导入单个表分区，其中每个分区都驻留在一个文件表空间文件中。

在下面的示例中，导入了一个四分区表的两个分区（`p2`和`p3`）。

1.  在目标实例上，创建一个与要导入分区的分区表定义相同的分区表（可以使用`SHOW CREATE TABLE`语法获取表定义）。如果表定义不匹配，则在尝试导入操作时会报告模式不匹配错误。

    ```sql
    mysql> USE test;
    mysql> CREATE TABLE t1 (i int) ENGINE = InnoDB PARTITION BY KEY (i) PARTITIONS 4;
    ```

    在`/*`datadir`*/test`目录中，每个分区都有一个`.ibd`文件。

    ```sql
    mysql> \! ls */path/to/datadir*/test/
    t1#p#p0.ibd  t1#p#p1.ibd  t1#p#p2.ibd t1#p#p3.ibd
    ```

1.  在目标实例上，丢弃要从源实例导入的分区。（在导入分区之前，必须从接收分区表中丢弃相应的分区。）

    ```sql
    mysql> ALTER TABLE t1 DISCARD PARTITION p2, p3 TABLESPACE;
    ```

    两个丢弃分区（`p2`和`p3`）的表空间`.ibd`文件从目标实例的`/*`datadir`*/test`目录中移除，留下以下文件：

    ```sql
    mysql> \! ls */path/to/datadir*/test/
    t1#p#p0.ibd  t1#p#p1.ibd
    ```

    注意

    当在子分区表上运行`ALTER TABLE ... DISCARD PARTITION ... TABLESPACE`时，允许使用分区和子分区表名称。指定分区名称时，该分区的子分区也包括在操作中。

1.  在源实例上，运行`FLUSH TABLES ... FOR EXPORT`使分区表静止。当表被静止时，只允许对表进行只读事务。

    ```sql
    mysql> USE test;
    mysql> FLUSH TABLES t1 FOR EXPORT;
    ```

    `FLUSH TABLES ... FOR EXPORT`确保将对命名表的更改刷新到磁盘，以便在实例运行时可以进行二进制表复制。运行`FLUSH TABLES ... FOR EXPORT`时，`InnoDB`为表模式目录中的每个表空间文件生成一个`.cfg`元数据文件。

    ```sql
    mysql> \! ls */path/to/datadir*/test/
    t1#p#p0.ibd  t1#p#p1.ibd  t1#p#p2.ibd t1#p#p3.ibd
    t1#p#p0.cfg  t1#p#p1.cfg  t1#p#p2.cfg t1#p#p3.cfg
    ```

    `.cfg`文件包含在导入操作期间用于模式验证的元数据。`FLUSH TABLES ... FOR EXPORT`只能在表上运行，而不能在单独的表分区上运行。

1.  从源实例模式目录复制分区`p2`和分区`p3`的`.ibd`和`.cfg`文件到目标实例模式目录。

    ```sql
    $> scp t1#p#p2.ibd t1#p#p2.cfg t1#p#p3.ibd t1#p#p3.cfg destination-server:*/path/to/datadir*/test
    ```

    在释放共享锁之前，必须复制`.ibd`和`.cfg`文件，如下一步骤所述。

    注意

    如果您从加密表空间导入分区，则`InnoDB`会生成一个`.cfg`元数据文件以及一个`.cfp`文件。`.cfp`文件必须与`.cfg`文件一起复制到目标实例。`.cfp`文件包含传输密钥和加密表空间密钥。在导入时，`InnoDB`使用传输密钥解密表空间密钥。有关更多信息，请参见第 17.13 节，“InnoDB 数据静态加密”。

1.  在源实例上，使用`UNLOCK TABLES`释放由`FLUSH TABLES ... FOR EXPORT`获取的锁：

    ```sql
    mysql> USE test;
    mysql> UNLOCK TABLES;
    ```

1.  在目标实例上，导入表分区`p2`和`p3`：

    ```sql
    mysql> USE test;
    mysql> ALTER TABLE t1 IMPORT PARTITION p2, p3 TABLESPACE;
    ```

    注意

    当在分区化表上运行`ALTER TABLE ... IMPORT PARTITION ... TABLESPACE`时，允许使用分区和子分区表名称。指定分区名称时，该分区的子分区将包含在操作中。

##### 限制

+   *可传输表空间*功能仅支持驻留在文件表空间中的表。不支持驻留在系统表空间或通用表空间中的表。共享表空间中的表无法静止。

+   不支持在具有`FULLTEXT`索引的表上运行`FLUSH TABLES ... FOR EXPORT`，因为无法刷新全文搜索辅助表。在导入具有`FULLTEXT`索引的表后，运行`OPTIMIZE TABLE`来重建`FULLTEXT`索引。或者，在导出操作之前删除`FULLTEXT`索引，并在目标实例上导入表后重新创建索引。

+   由于`.cfg`元数据文件的限制，当导入分区表时，不会报告分区类型或分区定义差异，但会报告列差异。

+   在 MySQL 8.0.19 之前，在表空间导入操作期间，索引键部分排序顺序信息不会存储到使用的`.cfg`元数据文件中。因此，假定索引键部分排序顺序为升序，这是默认值。因此，如果一个表在导入操作中定义为具有 DESC 索引键部分排序顺序，而另一个表没有，则记录可能以意外的顺序排序。解决方法是删除并重新创建受影响的索引。有关索引键部分排序顺序的信息，请参见第 15.1.15 节，“CREATE INDEX Statement”。

    MySQL 8.0.19 中更新了`.cfg`文件格式，包括索引键部分排序顺序信息。上述问题不会影响 MySQL 8.0.19 服务器实例或更高版本之间的导入操作。

##### 使用说明

+   除了包含立即添加或删除列的表之外，`ALTER TABLE ... IMPORT TABLESPACE`在导入表时不需要`.cfg`元数据文件。然而，在没有使用`.cfg`文件导入时不会执行元数据检查，并会发出类似以下警告：

    ```sql
    Message: InnoDB: IO Read error: (2, No such file or directory) Error opening '.\
    test\t.cfg', will attempt to import without schema verification
    1 row in set (0.00 sec)
    ```

    只有在不期望出现模式不匹配且表不包含任何立即添加或删除列时，才应考虑在没有`.cfg`元数据文件的情况下导入表。在无法访问元数据的崩溃恢复场景中，无需`.cfg`文件即可导入可能是有用的。

    尝试使用`ALGORITHM=INSTANT`导入包含已添加或删除列的表而没有使用`.cfg`文件可能导致未定义的行为。

+   在 Windows 上，`InnoDB`在内部以小写存储数据库、表空间和表名。为避免在 Linux 和 Unix 等区分大小写的操作系统上出现导入问题，请使用小写名称创建所有数据库、表空间和表。确保名称以小写创建的一种便捷方法是在初始化服务器之前将`lower_case_table_names`设置为 1。（禁止使用与服务器初始化时使用的设置不同的`lower_case_table_names`设置启动服务器。）

    ```sql
    [mysqld]
    lower_case_table_names=1
    ```

+   在子分区表上运行`ALTER TABLE ... DISCARD PARTITION ... TABLESPACE`和`ALTER TABLE ... IMPORT PARTITION ... TABLESPACE`时，允许使用分区和子分区表名称。指定分区名称时，该分区的子分区将包含在操作中。

##### 内部结构

以下信息描述了在表导入过程中写入错误日志的内部和消息。

当在目标实例上运行`ALTER TABLE ... DISCARD TABLESPACE`时：

+   表以 X 模式被锁定。

+   表空间从表中分离。

当在源实例上运行`FLUSH TABLES ... FOR EXPORT`时：

+   用于导出的表被以共享模式锁定。

+   清理协调器线程被停止。

+   脏页被同步到磁盘。

+   表元数据被写入二进制`.cfg`文件。

该操作的预期错误日志消息：

```sql
[Note] InnoDB: Sync to disk of '"test"."t1"' started.
[Note] InnoDB: Stopping purge
[Note] InnoDB: Writing table metadata to './test/t1.cfg'
[Note] InnoDB: Table '"test"."t1"' flushed to disk
```

当在源实例上运行`UNLOCK TABLES`时：

+   二进制`.cfg`文件被删除。

+   被导入的表或表的共享锁被释放，并且清理协调器线程被重新启动。

该操作的预期错误日志消息：

```sql
[Note] InnoDB: Deleting the meta-data file './test/t1.cfg'
[Note] InnoDB: Resuming purge
```

当在目标实例上运行`ALTER TABLE ... IMPORT TABLESPACE`时，导入算法对每个被导入的表空间执行以下操作：

+   检查每个表空间页是否损坏。

+   更新每个页面上的空间 ID 和日志序列号（LSN）。

+   标志被验证并且头页的 LSN 被更新。

+   B 树页被更新。

+   页面状态被设置为脏，以便写入磁盘。

该操作的预期错误日志消息：

```sql
[Note] InnoDB: Importing tablespace for table 'test/t1' that was exported
from host '*host_name*'
[Note] InnoDB: Phase I - Update all pages
[Note] InnoDB: Sync to disk
[Note] InnoDB: Sync to disk - done!
[Note] InnoDB: Phase III - Flush changes to disk
[Note] InnoDB: Phase IV - Flush complete
```

注意

你可能还会收到一个警告，表空间被丢弃了（如果你丢弃了目标表的表空间），以及一个消息说明由于缺少`.ibd`文件而无法计算统计信息：

```sql
[Warning] InnoDB: Table "test"."t1" tablespace is set as discarded.
7f34d9a37700 InnoDB: cannot calculate statistics for table
"test"."t1" because the .ibd file is missing. For help, please refer to
http://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting.html
```
