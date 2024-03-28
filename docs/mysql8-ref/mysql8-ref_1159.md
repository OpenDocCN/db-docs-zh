> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-innodb-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/using-innodb-tables.html)

#### 17.6.1.1 创建 InnoDB 表

使用`CREATE TABLE`语句创建`InnoDB`表；例如：

```sql
CREATE TABLE t1 (a INT, b CHAR (20), PRIMARY KEY (a)) ENGINE=InnoDB;
```

当`InnoDB`被定义为默认存储引擎时（默认情况下是这样），不需要`ENGINE=InnoDB`子句。但是，如果要在默认存储引擎不是`InnoDB`或未知的另一个 MySQL 服务器实例上重放`CREATE TABLE`语句，则`ENGINE`子句很有用。您可以通过发出以下语句来确定 MySQL 服务器实例上的默认存储引擎：

```sql
mysql> SELECT @@default_storage_engine;
+--------------------------+
| @@default_storage_engine |
+--------------------------+
| InnoDB                   |
+--------------------------+
```

`InnoDB`表默认在每个表的表空间中创建。要在`InnoDB`系统表空间中创建`InnoDB`表，请在创建表之前禁用`innodb_file_per_table`变量。要在一般表空间中创建`InnoDB`表，请使用`CREATE TABLE ... TABLESPACE`语法。更多信息，请参见第 17.6.3 节，“表空间”。

##### 行格式

`InnoDB`表的行格式决定了其行在磁盘上的物理存储方式。`InnoDB`支持四种行格式，每种都具有不同的存储特性。支持的行格式包括`REDUNDANT`、`COMPACT`、`DYNAMIC`和`COMPRESSED`。`DYNAMIC`行格式是默认的。有关行格式特性的信息，请参见第 17.10 节，“InnoDB 行格式”。

`innodb_default_row_format`变量定义了默认行格式。表的行格式也可以在`CREATE TABLE`或`ALTER TABLE`语句中使用`ROW_FORMAT`表选项显式定义。请参见定义表的行格式。

##### 主键

建议为创建的每个表定义一个主键。在选择主键列时，请选择具有以下特征的列：

+   被最重要查询引用的列。

+   从不留空的列。

+   从不具有重复值的列。

+   一旦插入后很少或几乎不会更改值的列。

例如，在包含有关人员信息的表中，您不会在`(firstname, lastname)`上创建主键，因为可能有多个人具有相同的姓名，姓名列可能为空，有时人们会更改他们的姓名。由于有这么多的约束条件，通常没有明显的列集可用作主键，因此您可以创建一个新的带有数字 ID 的列，作为主键的全部或部分。您可以声明一个自增列，以便在插入行时自动填入升序值：

```sql
# The value of ID can act like a pointer between related items in different tables.
CREATE TABLE t5 (id INT AUTO_INCREMENT, b CHAR (20), PRIMARY KEY (id));

# The primary key can consist of more than one column. Any autoinc column must come first.
CREATE TABLE t6 (id INT AUTO_INCREMENT, a INT, b CHAR (20), PRIMARY KEY (id,a));
```

有关自增列的更多信息，请参阅第 17.6.1.6 节，“AUTO_INCREMENT Handling in InnoDB”。

虽然表在没有定义主键的情况下可以正常工作，但主键涉及到性能的许多方面，对于任何大型或经常使用的表来说，主键是一个关键的设计方面。建议您在`CREATE TABLE`语句中始终指定主键。如果您创建表、加载数据，然后运行`ALTER TABLE`以后添加主键，那么该操作比在创建表时定义主键要慢得多。有关主键的更多信息，请参阅第 17.6.2.1 节，“Clustered and Secondary Indexes”。

##### 查看 InnoDB 表属性

要查看`InnoDB`表的属性，请发出`SHOW TABLE STATUS`语句：

```sql
mysql> SHOW TABLE STATUS FROM test LIKE 't%' \G;
*************************** 1\. row ***************************
           Name: t1
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2021-02-18 12:18:28
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment:
```

关于`SHOW TABLE STATUS`输出的信息，请参阅第 15.7.7.38 节，“SHOW TABLE STATUS Statement”。

您还可以通过查询`InnoDB`信息模式系统表来访问`InnoDB`表属性：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME='test/t1' \G
*************************** 1\. row ***************************
     TABLE_ID: 1144
         NAME: test/t1
         FLAG: 33
       N_COLS: 5
        SPACE: 30
   ROW_FORMAT: Dynamic
ZIP_PAGE_SIZE: 0
   SPACE_TYPE: Single
 INSTANT_COLS: 0
```

有关更多信息，请参阅第 17.15.3 节，“InnoDB INFORMATION_SCHEMA Schema Object Tables”。
