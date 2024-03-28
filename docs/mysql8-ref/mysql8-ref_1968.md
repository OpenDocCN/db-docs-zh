# 28.4.22 The INFORMATION_SCHEMA INNODB_SESSION_TEMP_TABLESPACES Table

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-session-temp-tablespaces-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-session-temp-tablespaces-table.html)

`INNODB_SESSION_TEMP_TABLESPACES`表提供有关用于内部和用户创建的临时表的会话临时表空间的元数据。此表在 MySQL 8.0.13 中添加。

`INNODB_SESSION_TEMP_TABLESPACES`表具有以下列：

+   `ID`

    进程或会话 ID。

+   `空间`

    表空间 ID。为会话临时表空间保留了 40 万个空间 ID 范围。每次服务器启动时都会重新创建会话临时表空间。在关闭服务器时，空间 ID 不会持久化，并且可能会被重用。

+   `路径`

    表空间数据文件路径。会话临时表空间具有`ibt`文件扩展名。

+   `大小`

    表空间的大小，以字节为单位。

+   `状态`

    表空间的状态。`ACTIVE`表示该表空间当前被会话使用。`INACTIVE`表示该表空间在可用会话临时表空间池中。

+   `目的`

    表空间的目的。`INTRINSIC`表示该表空间用于优化内部临时表，由优化器使用。`SLAVE`表示该表空间用于在复制从库上存储用户创建的临时表。`USER`表示该表空间用于用户创建的临时表。`NONE`表示该表空间未被使用。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_SESSION_TEMP_TABLESPACES;
+----+------------+----------------------------+-------+----------+-----------+
| ID | SPACE      | PATH                       | SIZE  | STATE    | PURPOSE   |
+----+------------+----------------------------+-------+----------+-----------+
|  8 | 4294566162 | ./#innodb_temp/temp_10.ibt | 81920 | ACTIVE   | INTRINSIC |
|  8 | 4294566161 | ./#innodb_temp/temp_9.ibt  | 98304 | ACTIVE   | USER      |
|  0 | 4294566153 | ./#innodb_temp/temp_1.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566154 | ./#innodb_temp/temp_2.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566155 | ./#innodb_temp/temp_3.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566156 | ./#innodb_temp/temp_4.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566157 | ./#innodb_temp/temp_5.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566158 | ./#innodb_temp/temp_6.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566159 | ./#innodb_temp/temp_7.ibt  | 81920 | INACTIVE | NONE      |
|  0 | 4294566160 | ./#innodb_temp/temp_8.ibt  | 81920 | INACTIVE | NONE      |
+----+------------+----------------------------+-------+----------+-----------+
```

#### 注意

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA` `COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。
