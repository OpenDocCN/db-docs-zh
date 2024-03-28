# 17.15.3 InnoDB INFORMATION_SCHEMA 模式对象表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-system-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-system-tables.html)

您可以使用`InnoDB` `INFORMATION_SCHEMA`表提取关于由`InnoDB`管理的模式对象的元数据。这些信息来自数据字典。传统上，您可以使用第 17.17 节“InnoDB 监视器”中的技术，设置`InnoDB`监视器并解析`SHOW ENGINE INNODB STATUS`语句的输出来获取此类型的信息。`InnoDB` `INFORMATION_SCHEMA`表接口允许您使用 SQL 查询这些数据。

`InnoDB` `INFORMATION_SCHEMA`模式对象表包括下面列出的表。

```sql
INNODB_DATAFILES
INNODB_TABLESTATS
INNODB_FOREIGN
INNODB_COLUMNS
INNODB_INDEXES
INNODB_FIELDS
INNODB_TABLESPACES
INNODB_TABLESPACES_BRIEF
INNODB_FOREIGN_COLS
INNODB_TABLES
```

表名反映��提供的数据类型：

+   `INNODB_TABLES`提供了关于`InnoDB`表的元数据。

+   `INNODB_COLUMNS`提供了关于`InnoDB`表列的元数据。

+   `INNODB_INDEXES`提供了关于`InnoDB`索引的元数据。

+   `INNODB_FIELDS`提供了关于`InnoDB`索引的关键列（字段）的元数据。

+   `INNODB_TABLESTATS`提供了关于`InnoDB`表的低级状态信息的视图，这些信息是从内存数据结构中派生的。

+   `INNODB_DATAFILES`提供了`InnoDB`文件表和通用表空间的数据文件路径信息。

+   `INNODB_TABLESPACES`提供了关于`InnoDB`文件表、通用表和撤销表空间的元数据。

+   `INNODB_TABLESPACES_BRIEF`提供了关于`InnoDB`表空间的部分元数据。

+   `INNODB_FOREIGN`提供了关于在`InnoDB`表上定义的外键的元数据。

+   `INNODB_FOREIGN_COLS`提供了关于在`InnoDB`表上定义的外键列的元数据。

`InnoDB` `INFORMATION_SCHEMA`模式对象表可以通过`TABLE_ID`、`INDEX_ID`和`SPACE`等字段进行连接，使您可以轻松检索要研究或监视的对象的所有可用数据。

参考`InnoDB` INFORMATION_SCHEMA 文档，了解每个表的列信息。

**示例 17.2 InnoDB INFORMATION_SCHEMA 模式对象表**

本示例使用一个简单的表（`t1`）和一个单一索引（`i1`）来演示在`InnoDB` `INFORMATION_SCHEMA`模式对象表中找到的元数据类型。

1.  创建一个测试数据库和表`t1`：

    ```sql
    mysql> CREATE DATABASE test;

    mysql> USE test;

    mysql> CREATE TABLE t1 (
           col1 INT,
           col2 CHAR(10),
           col3 VARCHAR(10))
           ENGINE = InnoDB;

    mysql> CREATE INDEX i1 ON t1(col1);
    ```

1.  创建表`t1`后，查询`INNODB_TABLES`以查找`test/t1`的元数据：

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME='test/t1' \G
    *************************** 1\. row ***************************
         TABLE_ID: 71
             NAME: test/t1
             FLAG: 1
           N_COLS: 6
            SPACE: 57
       ROW_FORMAT: Compact
    ZIP_PAGE_SIZE: 0
     INSTANT_COLS: 0
    ```

    表`t1`的`TABLE_ID`为 71。`FLAG`字段提供有关表格式和存储特性的位级信息。共有六列，其中三列是由`InnoDB`创建的隐藏列（`DB_ROW_ID`、`DB_TRX_ID`和`DB_ROLL_PTR`）。表的`SPACE`的 ID 为 57（值为 0 表示表位于系统表空间中）。`ROW_FORMAT`为 Compact。`ZIP_PAGE_SIZE`仅适用于具有`Compressed`行格式的表。`INSTANT_COLS`显示在使用`ALTER TABLE ... ADD COLUMN`添加第一个即时列之前表中的列数。

1.  使用`INNODB_TABLES`中的`TABLE_ID`信息，查询`INNODB_COLUMNS`表以获取有关表列的信息。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_COLUMNS where TABLE_ID = 71\G
    *************************** 1\. row ***************************
         TABLE_ID: 71
             NAME: col1
              POS: 0
            MTYPE: 6
           PRTYPE: 1027
              LEN: 4
      HAS_DEFAULT: 0
    DEFAULT_VALUE: NULL
    *************************** 2\. row ***************************
         TABLE_ID: 71
             NAME: col2
              POS: 1
            MTYPE: 2
           PRTYPE: 524542
              LEN: 10
      HAS_DEFAULT: 0
    DEFAULT_VALUE: NULL
    *************************** 3\. row ***************************
         TABLE_ID: 71
             NAME: col3
              POS: 2
            MTYPE: 1
           PRTYPE: 524303
              LEN: 10
      HAS_DEFAULT: 0
    DEFAULT_VALUE: NULL
    ```

    除了`TABLE_ID`和列`NAME`之外，`INNODB_COLUMNS`还提供每列的序数位置（从 0 开始递增顺序），列`MTYPE`或“主类型”（6 = INT，2 = CHAR，1 = VARCHAR），`PRTYPE`或“精确类型”（一个二进制值，其中的位表示 MySQL 数据类型、字符集代码和可空性），以及列长度（`LEN`）。`HAS_DEFAULT`和`DEFAULT_VALUE`列仅适用于使用`ALTER TABLE ... ADD COLUMN`立即添加的列。

1.  再次使用`INNODB_TABLES`中的`TABLE_ID`信息，查询`INNODB_INDEXES`以获取与表`t1`关联的索引信息。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_INDEXES WHERE TABLE_ID = 71 \G
    *************************** 1\. row ***************************
           INDEX_ID: 111
               NAME: GEN_CLUST_INDEX
           TABLE_ID: 71
               TYPE: 1
           N_FIELDS: 0
            PAGE_NO: 3
              SPACE: 57
    MERGE_THRESHOLD: 50
    *************************** 2\. row ***************************
           INDEX_ID: 112
               NAME: i1
           TABLE_ID: 71
               TYPE: 0
           N_FIELDS: 1
            PAGE_NO: 4
              SPACE: 57
    MERGE_THRESHOLD: 50
    ```

    `INNODB_INDEXES`返回两个索引的数据。第一个索引是`GEN_CLUST_INDEX`，如果表没有用户定义的聚簇索引，则`InnoDB`会创建一个聚簇索引。第二个索引（`i1`）是用户定义的二级索引。

    `INDEX_ID`是一个在实例中所有数据库中唯一的索引标识符。`TABLE_ID`标识与索引关联的表。索引`TYPE`值表示索引类型（1 = 聚簇索引，0 = 二级索引）。`N_FILEDS`值是组成索引的字段数。`PAGE_NO`是索引 B 树的根页号，`SPACE`是索引所在的表空间的 ID。非零值表示索引不位于系统表空间中。`MERGE_THRESHOLD`定义了索引页中数据量的百分比阈值。如果索引页中的数据量低于此值（默认为 50%），当删除行或通过更新操作缩短行时，`InnoDB`会尝试将索引页与相邻的索引页合并。

1.  使用`INNODB_INDEXES`中的`INDEX_ID`信息，查询`INNODB_FIELDS`以获取索引`i1`的字段信息。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FIELDS where INDEX_ID = 112 \G
    *************************** 1\. row ***************************
    INDEX_ID: 112
        NAME: col1
         POS: 0
    ```

    `INNODB_FIELDS`提供索引字段的`NAME`和其在索引中的序号位置。如果索引（i1）是在多个字段上定义的，`INNODB_FIELDS`将为每个索引字段提供元数据。

1.  使用`INNODB_TABLES`中的`SPACE`信息，查询`INNODB_TABLESPACES`表以获取有关表的表空间的信息。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE SPACE = 57 \G
    *************************** 1\. row ***************************
              SPACE: 57
              NAME: test/t1
              FLAG: 16417
        ROW_FORMAT: Dynamic
         PAGE_SIZE: 16384
     ZIP_PAGE_SIZE: 0
        SPACE_TYPE: Single
     FS_BLOCK_SIZE: 4096
         FILE_SIZE: 114688
    ALLOCATED_SIZE: 98304
    AUTOEXTEND_SIZE: 0
    SERVER_VERSION: 8.0.23
     SPACE_VERSION: 1
        ENCRYPTION: N
             STATE: normal
    ```

    除了表空间的`SPACE` ID 和关联表的`NAME`之外，`INNODB_TABLESPACES`提供表空间`FLAG`数据，这是关于表空间格式和存储特性的位级信息。还提供了表空间`ROW_FORMAT`，`PAGE_SIZE`以及其他几个表空间元数据项。

1.  再次使用`INNODB_TABLES`中的`SPACE`信息，查询`INNODB_DATAFILES`以获取表空间数据文件的位置。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_DATAFILES WHERE SPACE = 57 \G
    *************************** 1\. row ***************************
    SPACE: 57
     PATH: ./test/t1.ibd
    ```

    数据文件位于 MySQL 的`data`目录下的`test`目录中。如果在 MySQL 数据目录之外的位置使用`CREATE TABLE`语句的`DATA DIRECTORY`子句创建了 file-per-table 表空间，则表空间`PATH`将是一个完全限定的目录路径。

1.  最后一步，向表`t1`（`TABLE_ID = 71`）插入一行数据，并查看`INNODB_TABLESTATS`表中的数据。此表中的数据由 MySQL 优化器用于计算在查询`InnoDB`表时使用哪个索引。这些信息来自内存数据结构。

    ```sql
    mysql> INSERT INTO t1 VALUES(5, 'abc', 'def');
    Query OK, 1 row affected (0.06 sec)

    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLESTATS where TABLE_ID = 71 \G
    *************************** 1\. row ***************************
             TABLE_ID: 71
                 NAME: test/t1
    STATS_INITIALIZED: Initialized
             NUM_ROWS: 1
     CLUST_INDEX_SIZE: 1
     OTHER_INDEX_SIZE: 0
     MODIFIED_COUNTER: 1
              AUTOINC: 0
            REF_COUNT: 1
    ```

    `STATS_INITIALIZED`字段指示表是否已收集统计信息。`NUM_ROWS`是表中当前估计的行数。`CLUST_INDEX_SIZE`和`OTHER_INDEX_SIZE`字段分别报告存储表的聚集索引和辅助索引的磁盘上的页面数。`MODIFIED_COUNTER`值显示被 DML 操作和外键级联操作修改的行数。`AUTOINC`值是任何自增操作即将发行的下一个数字。在表`t1`上没有定义自增列，因此该值为 0。`REF_COUNT`值是一个计数器。当计数器达到 0 时，表示表元数据可以从表缓存中驱逐。

**示例 17.3 外键 INFORMATION_SCHEMA 模式对象表**

`INNODB_FOREIGN`和`INNODB_FOREIGN_COLS`表提供有关外键关系的数据。此示例使用具有外键关系的父表和子表来演示在`INNODB_FOREIGN`和`INNODB_FOREIGN_COLS`表中找到的数据。

1.  创建具有父表和子表的测试数据库：

    ```sql
    mysql> CREATE DATABASE test;

    mysql> USE test;

    mysql> CREATE TABLE parent (id INT NOT NULL,
           PRIMARY KEY (id)) ENGINE=INNODB;

    mysql> CREATE TABLE child (id INT, parent_id INT,
           INDEX par_ind (parent_id),
           CONSTRAINT fk1
           FOREIGN KEY (parent_id) REFERENCES parent(id)
           ON DELETE CASCADE) ENGINE=INNODB;
    ```

1.  创建父表和子表后，查询`INNODB_FOREIGN`并找到`test/child`和`test/parent`外键关系的外键数据：

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FOREIGN \G
    *************************** 1\. row ***************************
          ID: test/fk1
    FOR_NAME: test/child
    REF_NAME: test/parent
      N_COLS: 1
        TYPE: 1
    ```

    元数据包括外键`ID`（`fk1`），该外键是在子表上定义的`CONSTRAINT`的名称。`FOR_NAME`是定义外键的子表的名称。`REF_NAME`是父表（“被引用”表）的名称。`N_COLS`是外键索引中的列数。`TYPE`是表示有关外键列的其他信息的位标志的数值。在这种情况下，`TYPE`值为 1，表示为外键指定了`ON DELETE CASCADE`选项。有关`TYPE`值的更多信息，请参阅`INNODB_FOREIGN`表定义。

1.  使用外键`ID`，查询`INNODB_FOREIGN_COLS`以查看有关外键列的数据。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FOREIGN_COLS WHERE ID = 'test/fk1' \G
    *************************** 1\. row ***************************
              ID: test/fk1
    FOR_COL_NAME: parent_id
    REF_COL_NAME: id
             POS: 0
    ```

    `FOR_COL_NAME`是子表中外键列的名称，`REF_COL_NAME`是父表中引用列的名称。`POS`值是外键索引中键字段的序数位置，从零开始。

**示例 17.4 连接 InnoDB INFORMATION_SCHEMA 模式对象表**

此示例演示了连接三个`InnoDB` `INFORMATION_SCHEMA`模式对象表（`INNODB_TABLES`、`INNODB_TABLESPACES`和`INNODB_TABLESTATS`）以收集有关员工示例数据库中表的文件格式、行格式、页面大小和索引大小信息。

以下表别名用于缩短查询字符串：

+   `INFORMATION_SCHEMA.INNODB_TABLES`：a

+   `INFORMATION_SCHEMA.INNODB_TABLESPACES`：b

+   `INFORMATION_SCHEMA.INNODB_TABLESTATS`：c

使用`IF()`控制流函数来处理压缩表。如果表被压缩，索引大小将使用`ZIP_PAGE_SIZE`而不是`PAGE_SIZE`来计算。`CLUST_INDEX_SIZE`和`OTHER_INDEX_SIZE`以字节报告，通过`1024*1024`除以以提供以兆字节（MB）为单位的索引大小。MB 值使用`ROUND()`函数四舍五入到零位小数。

```sql
mysql> SELECT a.NAME, a.ROW_FORMAT,
        @page_size :=
         IF(a.ROW_FORMAT='Compressed',
          b.ZIP_PAGE_SIZE, b.PAGE_SIZE)
          AS page_size,
         ROUND((@page_size * c.CLUST_INDEX_SIZE)
          /(1024*1024)) AS pk_mb,
         ROUND((@page_size * c.OTHER_INDEX_SIZE)
          /(1024*1024)) AS secidx_mb
       FROM INFORMATION_SCHEMA.INNODB_TABLES a
       INNER JOIN INFORMATION_SCHEMA.INNODB_TABLESPACES b on a.NAME = b.NAME
       INNER JOIN INFORMATION_SCHEMA.INNODB_TABLESTATS c on b.NAME = c.NAME
       WHERE a.NAME LIKE 'employees/%'
       ORDER BY a.NAME DESC;
+------------------------+------------+-----------+-------+-----------+
| NAME                   | ROW_FORMAT | page_size | pk_mb | secidx_mb |
+------------------------+------------+-----------+-------+-----------+
| employees/titles       | Dynamic    |     16384 |    20 |        11 |
| employees/salaries     | Dynamic    |     16384 |    93 |        34 |
| employees/employees    | Dynamic    |     16384 |    15 |         0 |
| employees/dept_manager | Dynamic    |     16384 |     0 |         0 |
| employees/dept_emp     | Dynamic    |     16384 |    12 |        10 |
| employees/departments  | Dynamic    |     16384 |     0 |         0 |
+------------------------+------------+-----------+-------+-----------+
```
