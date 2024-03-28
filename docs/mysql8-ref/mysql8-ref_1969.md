# 28.4.23 INFORMATION_SCHEMA INNODB_TABLES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tables-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-tables-table.html)

`INNODB_TABLES`表提供关于`InnoDB`表的元数据。

有关相关用法信息和示例，请参见第 17.15.3 节，“InnoDB INFORMATION_SCHEMA 模式对象表”。

`INNODB_TABLES`表具有以下列：

+   `TABLE_ID`

    `InnoDB`表的标识符。此值在实例中的所有数据库中是唯一的。

+   `NAME`

    表的名称，前面加上适当的模式（数据库）名称（例如，`test/t1`）。数据库和用户表的名称与它们最初定义时的大小写相同，可能受到`lower_case_table_names`设置的影响。

+   `FLAG`

    代表表格式和存储特性的位级信息的数值。

+   `N_COLS`

    表中的列数。报告的数字包括`InnoDB`创建的三个隐藏列（`DB_ROW_ID`、`DB_TRX_ID`和`DB_ROLL_PTR`）。报告的数字还包括虚拟生成列（如果存在）。

+   `SPACE`

    表所在表空间的标识符。0 表示`InnoDB`系统表空间。任何其他数字表示文件表空间或通用表空间。在`TRUNCATE TABLE`语句之后，此标识符保持不变。对于文件表空间，此标识符在实例中的所有数据库中对表是唯一的。

+   `ROW_FORMAT`

    表的行格式（`Compact`、`Redundant`、`Dynamic`或`Compressed`）。

+   `ZIP_PAGE_SIZE`

    压缩页大小。仅适用于行格式为`Compressed`的表。

+   `SPACE_TYPE`

    表所属的表空间类型。可能的值包括系统表空间的`System`，通用表空间的`General`和每个表的文件表空间的`Single`。使用`CREATE TABLE`或`ALTER TABLE` `TABLESPACE=innodb_system`分配给系统表空间的表具有`SPACE_TYPE`为`General`。有关更多信息，请参见`CREATE TABLESPACE`。

+   `INSTANT_COLS`

    在使用`ALTER TABLE ... ADD COLUMN`和`ALGORITHM=INSTANT`添加第一个瞬时列之前存在的列数。自 MySQL 8.0.29 起，不再使用此列，但继续显示在 MySQL 8.0.29 之前瞬时添加列的表的信息。

+   `TOTAL_ROW_VERSIONS`

    表的行版本数量。初始值为 0。通过增加或删除列的`ALTER TABLE ... ALGORITHM=INSTANT`操作来递增该值。当具有瞬时添加或删除列的表由于表重建的`ALTER TABLE`或`OPTIMIZE TABLE`操作而重建时，该值将重置为 0。有关更多信息，请参见列操作。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE TABLE_ID = 214\G
*************************** 1\. row ***************************
          TABLE_ID: 1064
              NAME: test/t1
              FLAG: 33
            N_COLS: 6
             SPACE: 3
        ROW_FORMAT: Dynamic
     ZIP_PAGE_SIZE: 0
        SPACE_TYPE: Single
      INSTANT_COLS: 0
TOTAL_ROW_VERSIONS: 3
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
