> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-moving-data-files-offline.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-moving-data-files-offline.html)

#### 17.6.3.6 在服务器离线时移动表空间文件

定义在启动时用于扫描表空间文件的`innodb_directories`变量支持在服务器离线时将表空间文件移动或恢复到新位置。在启动过程中，发现的表空间文件将被用于数据字典中引用的文件，并且数据字典将被更新以引用已重新定位的文件。如果扫描发现重复的表空间文件，则启动将失败，并显示错误，指示找到了相同表空间 ID 的多个文件。

由`innodb_data_home_dir`、`innodb_undo_directory`和`datadir`变量定义的目录会自动附加到`innodb_directories`参数值中。这些目录在启动时会被扫描，无论是否明确指定了`innodb_directories`设置。这些目录的隐式添加允许移动系统表空间文件、数据目录或撤销表空间文件，而无需配置`innodb_directories`设置。但是，当目录发生变化时，必须更新设置。例如，在重新定位数据目录后，必须在重新启动服务器之前更新`--datadir`设置。

`innodb_directories`变量可以在启动命令或 MySQL 选项文件中指定。由于分号（;）被一些命令解释器解释为特殊字符，因此在参数值周围使用引号。 （例如，Unix shell 将其视为命令终止符。）

启动命令：

```sql
mysqld --innodb-directories="*directory_path_1*;*directory_path_2*"
```

MySQL 选项文件：

```sql
[mysqld]
innodb_directories="*directory_path_1*;*directory_path_2*"
```

以下过程适用于移动单个 file-per-table 和 general tablespace 文件、system tablespace 文件、undo tablespace 文件或数据目录。在移动文件或目录之前，请查看以下使用说明。

1.  停止服务器。

1.  将表空间文件或目录移动到所需位置。

1.  使`InnoDB`知道新目录。

    +   如果移动单个 file-per-table 或通用表空间文件，请将未知目录添加到`innodb_directories`的值中。

        +   由`innodb_data_home_dir`、`innodb_undo_directory`和`datadir`变量定义的目录会自动附加到`innodb_directories`参数值中，因此无需指定这些目录。

        +   一个 file-per-table 表空间文件只能移动到与模式同名的目录中。例如，如果`actor`表属于`sakila`模式，则`actor.ibd`数据文件只能移动到名为`sakila`的目录中。

        +   通用表空间文件不能移动到数据目录或数据目录的子目录中。

    +   如果移动系统表空间文件、撤销表空间或数据目录，请根据需要更新`innodb_data_home_dir`、`innodb_undo_directory`和`datadir`设置。

1.  重新启动服务器。

##### 使用说明

+   不能在`innodb_directories`参数值中使用通配符表达式。

+   `innodb_directories`扫描还会遍历指定目录的子目录。重复的目录和子目录将从要扫描的目录列表中丢弃。

+   `innodb_directories`支持移动`InnoDB`表空间文件。不支持移动属于`InnoDB`以外存储引擎的文件。当移动整个数据目录时，此限制也适用。

+   `innodb_directories`支持在将文件移动到扫描目录时重命名表空间文件。它还支持将表空间文件移动到其他支持的操作系统。

+   在将表空间文件移动到不同操作系统时，请确保表空间文件名不包含目标系统上具有特殊含义或特殊含义的字符。

+   将数据目录从 Windows 操作系统移动到 Linux 操作系统时，请修改二进制日志文件路径在二进制日志索引文件中使用反斜杠而不是正斜杠。默认情况下，二进制日志索引文件与二进制日志文件具有相同的基本名称，扩展名为'`.index`'。二进制日志索引文件的位置由`--log-bin`定义。默认位置是数据目录。

+   如果将表空间文件移动到不同操作系统会引入跨平台复制，那么数据库管理员有责任确保包含特定平台目录的 DDL 语句的正确复制。允许指定目录的语句包括`CREATE TABLE ... DATA DIRECTORY`和`CREATE TABLESPACE ... ADD DATAFILE`。

+   将使用绝对路径或位于数据目录之外的位置创建的文件-每表和通用表空间的目录添加到`innodb_directories`设置中。否则，在恢复过程中，`InnoDB`将无法定位这些文件。有关更多信息，请参阅崩溃恢复期间的表空间发现。

    要查看表空间文件位置，请查询信息模式`FILES`表：

    ```sql
    mysql> SELECT TABLESPACE_NAME, FILE_NAME FROM INFORMATION_SCHEMA.FILES \G
    ```
