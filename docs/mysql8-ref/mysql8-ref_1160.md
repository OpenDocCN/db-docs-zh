> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-create-table-external.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-create-table-external.html)

#### 17.6.1.2 创建外部表

创建`InnoDB`表的外部原因有很多；也就是说，在数据目录之外创建表。这些原因可能包括空间管理、I/O 优化，或者将表放在具有特定性能或容量特征的存储设备上，例如。

`InnoDB`支持以下方法来外部创建表：

+   使用 DATA DIRECTORY 子句

+   使用 CREATE TABLE ... TABLESPACE 语法

+   在外部通用表空间中创建表

##### 使用 DATA DIRECTORY 子句

您可以通过在 `CREATE TABLE` 语句中指定 `DATA DIRECTORY` 子句来在外部目录中创建一个 `InnoDB` 表。

```sql
CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '*/external/directory*';
```

`DATA DIRECTORY` 子句支持在每表表空间中创建的表。当 `innodb_file_per_table` 变量启用时（默认情况下启用），表会隐式地在每表表空间中创建。

```sql
mysql> SELECT @@innodb_file_per_table;
+-------------------------+
| @@innodb_file_per_table |
+-------------------------+
|                       1 |
+-------------------------+
```

有关每表表空间的更多信息，请参见第 17.6.3.2 节，“每表表空间”。

当您在 `CREATE TABLE` 语句中指定 `DATA DIRECTORY` 子句时，表的数据文件（`*`table_name`*.ibd`）将在指定目录下的模式目录中创建。

从 MySQL 8.0.21 开始，使用 `DATA DIRECTORY` 子句在数据目录之外创建的表和表分区受到 `InnoDB` 知道的目录的限制。此要求允许数据库管理员控制表空间数据文件的创建位置，并确保在恢复期间可以找到数据文件（请参见崩溃恢复期间的表空间发现）。已知目录是由 `datadir`、`innodb_data_home_dir` 和 `innodb_directories` 变量定义的那些目录。您可以使用以下语句来检查这些设置：

```sql
mysql> SELECT @@datadir,@@innodb_data_home_dir,@@innodb_directories;
```

如果要使用的目录未知，请在创建表之前将其添加到`innodb_directories`设置中。`innodb_directories`变量是只读的。配置它需要重新启动服务器。有关设置系统变量的一般信息，请参见第 7.1.9 节，“使用系统变量”。

以下示例演示了使用`DATA DIRECTORY`子句在外部目录中创建表。假定`innodb_file_per_table`变量已启用，并且该目录为`InnoDB`所知。

```sql
mysql> USE test;
Database changed

mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '*/external/directory*';

# MySQL creates the table's data file in a schema directory
# under the external directory

$> cd /external/directory/test
$> ls
t1.ibd
```

###### 使用说明：

+   MySQL 最初会保持表空间数据文件打开，阻止您卸载设备，但如果服务器繁忙，可能最终会关闭文件。请注意不要在 MySQL 运行时意外卸载外部设备，或在设备断开连接时启动 MySQL。当相关数据文件丢失时尝试访问表会导致需要服务器重启的严重错误。

    如果在预期路径中找不到数据文件，服务器重启可能会失败。在这种情况下，您可以从备份中恢复表空间数据文件，或删除表以从数据字典中删除有关其信息。

+   在将表放在 NFS 挂载的卷上之前，请查看使用 NFS 与 MySQL 中概述的潜在问题。

+   如果使用 LVM 快照、文件复制或其他基于文件的机制来备份表的数据文件，请始终首先使用`FLUSH TABLES ... FOR EXPORT`语句，以确保所有在内存中缓冲的更改被刷新到磁盘上，然后再进行备份。

+   使用`DATA DIRECTORY`子句在外部目录中创建表是使用符号链接的替代方法，`InnoDB`不支持。

+   在源和副本位于同一主机的复制环境中不支持`DATA DIRECTORY`子句。`DATA DIRECTORY`子句需要完整的目录路径。在这种情况下复制路径会导致源和副本在相同位置创建表。

+   截至 MySQL 8.0.21，无法再在撤销表空间目录（`innodb_undo_directory`）中创建文件-每表表空间中的表，除非该目录为`InnoDB`所知。已知目录是由`datadir`、`innodb_data_home_dir`和`innodb_directories`变量定义的目录。

##### 使用 CREATE TABLE ... TABLESPACE 语法

`CREATE TABLE ... TABLESPACE`语法可以与`DATA DIRECTORY`子句结合使用，以在外部目录中创建表。为此，请将`innodb_file_per_table`指定为表空间名称。

```sql
mysql> CREATE TABLE t2 (c1 INT PRIMARY KEY) TABLESPACE = innodb_file_per_table
       DATA DIRECTORY = '/external/directory';
```

此方法仅支持在每个表的文件表空间中创建的表，但不需要启用`innodb_file_per_table`变量。在其他方面，此方法与上述描述的`CREATE TABLE ... DATA DIRECTORY`方法等效。相同的使用说明适用。

##### 在外部通用表空间中创建表

您可以在外部目录中的通用表空间中创建表。

+   有关在外部目录中创建通用表空间的信息，请参见创建通用表空间。

+   有关在通用表空间中创建表的信息，请参见将表添加到通用表空间。
