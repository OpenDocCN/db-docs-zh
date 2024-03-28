# 10.4.7 表列数和行大小的限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/column-count-limit.html`](https://dev.mysql.com/doc/refman/8.0/en/column-count-limit.html)

本节描述了表中列数和单个行大小的限制。

+   列数限制

+   行大小限制

#### 列数限制

MySQL 每个表的列数有 4096 列的硬限制，但对于给定表，实际最大值可能会更少。确切的列限制取决于几个因素：

+   表的最大行大小限制了列的数量（可能也是大小），因为所有列的总长度不能超过这个大小。参见行大小限制。

+   单个列的存储要求限制了适合于给定最大行大小的列数。某些数据类型的存储要求取决于诸如存储引擎、存储格式和字符集等因素。参见第 13.7 节，“数据类型存储要求”。

+   存储引擎可能会施加额外的限制，限制表列数。例如，`InnoDB`每个表有 1017 列的限制。参见第 17.22 节，“InnoDB 限制”。有关其他存储引擎的信息，请参见第十八章，“替代存储引擎”。

+   功能键部分（参见第 15.1.15 节，“CREATE INDEX 语句”）被实现为隐藏的虚拟生成的存储列，因此表索引中的每个功能键部分都计入表总列限制。

#### 行大小限制

给定表的最大行大小由几个因素决定：

+   MySQL 表的内部表示具有最大行大小限制为 65,535 字节，即使存储引擎能够支持更大的行。`BLOB`和`TEXT`列仅对行大小限制贡献 9 到 12 字节，因为它们的内容与行的其余部分分开存储。

+   `InnoDB`表的最大行大小，适用于数据库页面内存储的数据，对于 4KB、8KB、16KB 和 32KB 的`innodb_page_size`设置，略小于半页。例如，默认 16KB 的`InnoDB`页面大小的最大行大小略小于 8KB。对于 64KB 页面，最大行大小略小于 16KB。参见第 17.22 节，“InnoDB 限制”。

    如果包含变长列的行超过`InnoDB`的最大行大小，`InnoDB`会选择将变长列存储在外部页外存储，直到行符合`InnoDB`行大小限制。存储在本地的变长列数据量因行格式而异。有关更多信息，请参见第 17.10 节，“InnoDB 行格式”。

+   不同的存储格式使用不同数量的页头和页尾数据，这会影响可用于行的存储量。

    +   有关`InnoDB`行格式的信息，请参见第 17.10 节，“InnoDB 行格式”。

    +   有关`MyISAM`存储格式的信息，请参见第 18.2.3 节，“MyISAM 表存储格式”。

##### 行大小限制示例

+   MySQL 的最大行大小限制为 65,535 字节，以下示例演示了`InnoDB`和`MyISAM`的情况。该限制是强制执行的，无论存储引擎如何，即使存储引擎可能支持更大的行。

    ```sql
    mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
           c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
           f VARCHAR(10000), g VARCHAR(6000)) ENGINE=InnoDB CHARACTER SET latin1;
    ERROR 1118 (42000): Row size too large. The maximum row size for the used
    table type, not counting BLOBs, is 65535\. This includes storage overhead,
    check the manual. You have to change some columns to TEXT or BLOBs
    ```

    ```sql
    mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
           c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
           f VARCHAR(10000), g VARCHAR(6000)) ENGINE=MyISAM CHARACTER SET latin1;
    ERROR 1118 (42000): Row size too large. The maximum row size for the used
    table type, not counting BLOBs, is 65535\. This includes storage overhead,
    check the manual. You have to change some columns to TEXT or BLOBs
    ```

    在以下`MyISAM`示例中，将列更改为`TEXT`可以避免 65,535 字节的行大小限制，并且允许操作成功，因为`BLOB`和`TEXT`列只对行大小贡献了 9 到 12 字节。

    ```sql
    mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
           c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
           f VARCHAR(10000), g TEXT(6000)) ENGINE=MyISAM CHARACTER SET latin1;
    Query OK, 0 rows affected (0.02 sec)
    ```

    对于`InnoDB`表，将列更改为`TEXT`可以避免 MySQL 的 65,535 字节行大小限制，并且`InnoDB`的变长列页外存储避免了`InnoDB`行大小限制。

    ```sql
    mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
           c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
           f VARCHAR(10000), g TEXT(6000)) ENGINE=InnoDB CHARACTER SET latin1;
    Query OK, 0 rows affected (0.02 sec)
    ```

+   变长列的存储包括长度字节，这些字节计入行大小。例如，`VARCHAR(255) CHARACTER SET utf8mb3`列需要两个字节来存储值的长度，因此每个值最多可以占用 767 字节。

    创建表`t1`的语句成功，因为列需要 32,765 + 2 字节和 32,766 + 2 字节，这在最大行大小为 65,535 字节的范围内：

    ```sql
    mysql> CREATE TABLE t1
           (c1 VARCHAR(32765) NOT NULL, c2 VARCHAR(32766) NOT NULL)
           ENGINE = InnoDB CHARACTER SET latin1;
    Query OK, 0 rows affected (0.02 sec)
    ```

    创建表`t2`的语句失败，因为虽然列长度在 65,535 字节的最大长度范围内，但需要额外的两个字节来记录长度，这导致行大小超过 65,535 字节：

    ```sql
    mysql> CREATE TABLE t2
           (c1 VARCHAR(65535) NOT NULL)
           ENGINE = InnoDB CHARACTER SET latin1;
    ERROR 1118 (42000): Row size too large. The maximum row size for the used
    table type, not counting BLOBs, is 65535\. This includes storage overhead,
    check the manual. You have to change some columns to TEXT or BLOBs
    ```

    将列长度减少到 65,533 或更少可以使语句成功。

    ```sql
    mysql> CREATE TABLE t2
           (c1 VARCHAR(65533) NOT NULL)
           ENGINE = InnoDB CHARACTER SET latin1;
    Query OK, 0 rows affected (0.01 sec)
    ```

+   对于`MyISAM`表，`NULL`列需要额外的空间来记录它们的值是否为`NULL`。每个`NULL`列需要额外的一位，向上取整到最近的字节。

    创建表`t3`的语句失败，因为`MyISAM`除了需要变长列长度字节所需的空间外，还需要为`NULL`列留出空间，导致行大小超过 65,535 字节：

    ```sql
    mysql> CREATE TABLE t3
           (c1 VARCHAR(32765) NULL, c2 VARCHAR(32766) NULL)
           ENGINE = MyISAM CHARACTER SET latin1;
    ERROR 1118 (42000): Row size too large. The maximum row size for the used
    table type, not counting BLOBs, is 65535\. This includes storage overhead,
    check the manual. You have to change some columns to TEXT or BLOBs
    ```

    有关`InnoDB` `NULL`列存储的信息，请参阅第 17.10 节“InnoDB 行格式”。

+   `InnoDB`限制行大小（在数据库页面内本地存储的数据）略小于 4KB、8KB、16KB 和 32KB `innodb_page_size`设置的一半数据库页面大小，并且略小于 64KB 页面的 16KB。

    创建表`t4`的语句失败，因为定义的列超过了 16KB `InnoDB`页面的行大小限制。

    ```sql
    mysql> CREATE TABLE t4 (
           c1 CHAR(255),c2 CHAR(255),c3 CHAR(255),
           c4 CHAR(255),c5 CHAR(255),c6 CHAR(255),
           c7 CHAR(255),c8 CHAR(255),c9 CHAR(255),
           c10 CHAR(255),c11 CHAR(255),c12 CHAR(255),
           c13 CHAR(255),c14 CHAR(255),c15 CHAR(255),
           c16 CHAR(255),c17 CHAR(255),c18 CHAR(255),
           c19 CHAR(255),c20 CHAR(255),c21 CHAR(255),
           c22 CHAR(255),c23 CHAR(255),c24 CHAR(255),
           c25 CHAR(255),c26 CHAR(255),c27 CHAR(255),
           c28 CHAR(255),c29 CHAR(255),c30 CHAR(255),
           c31 CHAR(255),c32 CHAR(255),c33 CHAR(255)
           ) ENGINE=InnoDB ROW_FORMAT=DYNAMIC DEFAULT CHARSET latin1;
    ERROR 1118 (42000): Row size too large (> 8126). Changing some columns to TEXT or BLOB may help.
    In current row format, BLOB prefix of 0 bytes is stored inline.
    ```
