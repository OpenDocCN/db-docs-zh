> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dict-obj-tree.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dict-obj-tree.html)

#### 25.6.16.25 The ndbinfo dict_obj_tree Table

`dict_obj_tree`表提供了来自`dict_obj_info`表的基于树的表信息视图。这主要用于测试，但在可视化`NDB`数据库对象的层次结构方面也很有用。

`dict_obj_tree`表包含以下列：

+   `type`

    `DICT`对象的类型；连接`dict_obj_types`以获取对象类型的名称

+   `id`

    对象标识符；与`dict_obj_info`表中的`id`列相同

    对于磁盘数据撤销日志文件和数据文件，这与信息模式`FILES`表中的`LOGFILE_GROUP_NUMBER`列中显示的值相同；对于撤销日志文件，这也与 ndbinfo `logbuffers`和`logspaces`表中`log_id`列显示的值相同

+   `name`

    对象的完全限定名称；与`dict_obj_info`表中的`fq_name`列相同

    对于表，其形式为`*`database_name`*/def/*`table_name`*`（与其*`parent_name`*`相同）；对于任何类型的索引，形式为`NDB$INDEX_*`index_id`*_CUSTOM`

+   `parent_type`

    此对象父对象的*`DICT`*对象类型；连接`dict_obj_types`以获取对象类型的名称

+   `parent_id`

    此对象父对象的标识符；与`dict_obj_info`表的`id`列相同

+   `parent_name`

    此对象父对象的完全限定名称；与`dict_obj_info`表的`fq_name`列相同

    对于表，其形式为`*`database_name`*/def/*`table_name`*`。对于索引，名称为`sys/def/*`table_id`*/*`index_name`*`。对于主键，为`sys/def/*`table_id`*/PRIMARY`，对于唯一键，为`sys/def/*`table_id`*/*`uk_name`*$unique`

+   `root_type`

    根对象的*`DICT`*对象类型；连接`dict_obj_types`以获取对象类型的名称

+   `root_id`

    根对象的标识符；与`dict_obj_info`表的`id`列相同

+   `root_name`

    根对象的完全限定名称；与`dict_obj_info`表的`fq_name`列相同

+   `level`

    层次结构中对象的级别

+   `path`

    *`NDB`*对象层次结构中对象的完整路径；对象之间用右箭头（表示为`->`）分隔，从左侧的根对象开始

+   `indented_name`

    带有右箭头（表示为`->`）前缀的`name`，前面有一些空格，这些空格对应于层次结构中对象的深度

`path`列对于在单行中获取给定`NDB`数据库对象的完整路径非常有用，而`indented_name`列可用于获取所需对象的完整层次结构信息的类似树状布局。

*示例*：假设存在一个名为`test`的数据库，并且在此数据库中不存在名为`t1`的现有表，请执行以下 SQL 语句：

```sql
CREATE TABLE test.t1 (
    a INT PRIMARY KEY,
    b INT,
    UNIQUE KEY(b)
)   ENGINE = NDB;
```

您可以使用此处显示的查询获取刚创建的表的路径：

```sql
mysql> SELECT path FROM ndbinfo.dict_obj_tree
 -> WHERE name LIKE 'test%t1';
+-------------+
| path        |
+-------------+
| test/def/t1 |
+-------------+
1 row in set (0.14 sec)
```

您可以使用表的路径作为查询中的根名称，查看此表的所有依赖对象的路径，类似于以下查询：

```sql
mysql> SELECT path FROM ndbinfo.dict_obj_tree
 -> WHERE root_name = 'test/def/t1';
+----------------------------------------------------------+
| path                                                     |
+----------------------------------------------------------+
| test/def/t1                                              |
| test/def/t1 -> sys/def/13/b                              |
| test/def/t1 -> sys/def/13/b -> NDB$INDEX_15_CUSTOM       |
| test/def/t1 -> sys/def/13/b$unique                       |
| test/def/t1 -> sys/def/13/b$unique -> NDB$INDEX_16_UI    |
| test/def/t1 -> sys/def/13/PRIMARY                        |
| test/def/t1 -> sys/def/13/PRIMARY -> NDB$INDEX_14_CUSTOM |
+----------------------------------------------------------+
7 rows in set (0.16 sec)
```

要获取具有所有依赖对象的`t1`表的分层视图，请执行类似于以下查询的查询，该查询选择具有`test/def/t1`作为其根对象名称的每个对象的缩进名称：

```sql
mysql> SELECT indented_name FROM ndbinfo.dict_obj_tree
 -> WHERE root_name = 'test/def/t1';
+----------------------------+
| indented_name              |
+----------------------------+
| test/def/t1                |
|   -> sys/def/13/b          |
|     -> NDB$INDEX_15_CUSTOM |
|   -> sys/def/13/b$unique   |
|     -> NDB$INDEX_16_UI     |
|   -> sys/def/13/PRIMARY    |
|     -> NDB$INDEX_14_CUSTOM |
+----------------------------+
7 rows in set (0.15 sec)
```

在处理磁盘数据表时，请注意，在此上下文中，表空间或日志文件组被视为根对象。这意味着您必须知道与给定表相关联的任何表空间或日志文件组的名称，或者从`SHOW CREATE TABLE`中获取此信息，然后查询`INFORMATION_SCHEMA.FILES`或类似的方式，如下所示：

```sql
mysql> SHOW CREATE TABLE test.dt_1\G
*************************** 1\. row ***************************
       Table: dt_1
Create Table: CREATE TABLE `dt_1` (
  `member_id` int unsigned NOT NULL AUTO_INCREMENT,
  `last_name` varchar(50) NOT NULL,
  `first_name` varchar(50) NOT NULL,
  `dob` date NOT NULL,
  `joined` date NOT NULL,
  PRIMARY KEY (`member_id`),
  KEY `last_name` (`last_name`,`first_name`)
) /*!50100 TABLESPACE `ts_1` STORAGE DISK */ ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)

mysql> SELECT DISTINCT TABLESPACE_NAME, LOGFILE_GROUP_NAME
 -> FROM INFORMATION_SCHEMA.FILES WHERE TABLESPACE_NAME='ts_1';
+-----------------+--------------------+
| TABLESPACE_NAME | LOGFILE_GROUP_NAME |
+-----------------+--------------------+
| ts_1            | lg_1               |
+-----------------+--------------------+
1 row in set (0.00 sec)
```

现在，您可以像这样获取有关表、表空间和日志文件组的分层信息：

```sql
mysql> SELECT indented_name FROM ndbinfo.dict_obj_tree
 -> WHERE root_name = 'test/def/dt_1';
+----------------------------+
| indented_name              |
+----------------------------+
| test/def/dt_1              |
|   -> sys/def/23/last_name  |
|     -> NDB$INDEX_25_CUSTOM |
|   -> sys/def/23/PRIMARY    |
|     -> NDB$INDEX_24_CUSTOM |
+----------------------------+
5 rows in set (0.15 sec)

mysql> SELECT indented_name FROM ndbinfo.dict_obj_tree
 -> WHERE root_name = 'ts_1';
+-----------------+
| indented_name   |
+-----------------+
| ts_1            |
|   -> data_1.dat |
|   -> data_2.dat |
+-----------------+
3 rows in set (0.17 sec)

mysql> SELECT indented_name FROM ndbinfo.dict_obj_tree
 -> WHERE root_name LIKE 'lg_1';
+-----------------+
| indented_name   |
+-----------------+
| lg_1            |
|   -> undo_1.log |
|   -> undo_2.log |
+-----------------+
3 rows in set (0.16 sec)
```

`dict_obj_tree`表是在 NDB 8.0.24 中添加的。
