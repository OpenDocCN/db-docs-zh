# 28.4.29 INFORMATION_SCHEMA INNODB_VIRTUAL 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-virtual-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-virtual-table.html)

`INNODB_VIRTUAL`表提供关于`InnoDB`虚拟生成列和虚拟生成列所基于的列的元数据。

每个虚拟生成列所基于的列在`INNODB_VIRTUAL`表中都会出现一行。

`INNODB_VIRTUAL`表具有以下列：

+   `TABLE_ID`

    表示与虚拟列关联的表的标识符；与`INNODB_TABLES.TABLE_ID`相同的值。

+   `POS`

    虚拟生成列的位置值。该值很大，因为它编码了列序号和序号位置。用于计算该值的公式使用位操作：

    ```sql
    ((*n*th virtual generated column for the InnoDB instance + 1) << 16)
    + the ordinal position of the virtual generated column
    ```

    例如，如果`InnoDB`实例中的第一个虚拟生成列是表中的第三列，则公式为`((0 + 1) << 16) + 2`。`InnoDB`实例中的第一个虚拟生成列始终为编号 0。作为表中的第三列，虚拟生成列的序号位置为 2。序号位置从 0 开始计数。

+   `BASE_POS`

    虚拟生成列所基于的列的序号位置。

#### 示例

```sql
mysql> CREATE TABLE `t1` (
         `a` int(11) DEFAULT NULL,
         `b` int(11) DEFAULT NULL,
         `c` int(11) GENERATED ALWAYS AS (a+b) VIRTUAL,
         `h` varchar(10) DEFAULT NULL
       ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_VIRTUAL
       WHERE TABLE_ID IN
         (SELECT TABLE_ID FROM INFORMATION_SCHEMA.INNODB_TABLES
          WHERE NAME LIKE "test/t1");
+----------+-------+----------+
| TABLE_ID | POS   | BASE_POS |
+----------+-------+----------+
|       98 | 65538 |        0 |
|       98 | 65538 |        1 |
+----------+-------+----------+
```

#### 注意

+   如果将常量值分配给虚拟生成列，如下表所示，列的条目不会出现在`INNODB_VIRTUAL`表中。要出现条目，虚拟生成列必须有基本列。

    ```sql
    CREATE TABLE `t1` (
      `a` int(11) DEFAULT NULL,
      `b` int(11) DEFAULT NULL,
      `c` int(11) GENERATED ALWAYS AS (5) VIRTUAL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
    ```

    但是，这样的列的元数据确实出现在`INNODB_COLUMNS`表中。

+   您必须拥有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA``COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
