# 17.15.8 从 INFORMATION_SCHEMA.FILES 检索 InnoDB 表空间元数据

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-files-table.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-files-table.html)

信息模式`FILES`表提供有关所有`InnoDB`表空间类型的元数据，包括 file-per-table tablespaces、general tablespaces、system tablespace、temporary table tablespaces 和 undo tablespaces（如果存在）。

本节提供了`InnoDB`特定的用法示例。有关信息模式`FILES`表提供的数据的更多信息，请参阅 Section 28.3.15, “INFORMATION_SCHEMA FILES 表”。

注意

`INNODB_TABLESPACES`和`INNODB_DATAFILES`表还提供有关`InnoDB`表空间的元数据，但数据仅限于文件表、一般表和撤销表空间。

此查询从与`InnoDB`表空间相关的信息模式`FILES`表的字段中检索有关`InnoDB`系统表空间的元数据。与`InnoDB`不相关的`FILES`列始终返回`NULL`，并且在查询中被排除。

```sql
mysql> SELECT FILE_ID, FILE_NAME, FILE_TYPE, TABLESPACE_NAME, FREE_EXTENTS,
       TOTAL_EXTENTS,  EXTENT_SIZE, INITIAL_SIZE, MAXIMUM_SIZE, AUTOEXTEND_SIZE, DATA_FREE, STATUS ENGINE
       FROM INFORMATION_SCHEMA.FILES WHERE TABLESPACE_NAME LIKE 'innodb_system' \G
*************************** 1\. row ***************************
        FILE_ID: 0
      FILE_NAME: ./ibdata1
      FILE_TYPE: TABLESPACE
TABLESPACE_NAME: innodb_system
   FREE_EXTENTS: 0
  TOTAL_EXTENTS: 12
    EXTENT_SIZE: 1048576
   INITIAL_SIZE: 12582912
   MAXIMUM_SIZE: NULL
AUTOEXTEND_SIZE: 67108864
      DATA_FREE: 4194304
         ENGINE: NORMAL
```

此查询检索`InnoDB`文件表和一般表空间的`FILE_ID`（相当于空间 ID）和`FILE_NAME`（包括路径信息）。文件表和一般表空间的文件扩展名为`.ibd`。

```sql
mysql> SELECT FILE_ID, FILE_NAME FROM INFORMATION_SCHEMA.FILES
       WHERE FILE_NAME LIKE '%.ibd%' ORDER BY FILE_ID;
 +---------+---------------------------------------+
 | FILE_ID | FILE_NAME                             |
 +---------+---------------------------------------+
 |       2 | ./mysql/plugin.ibd                    |
 |       3 | ./mysql/servers.ibd                   |
 |       4 | ./mysql/help_topic.ibd                |
 |       5 | ./mysql/help_category.ibd             |
 |       6 | ./mysql/help_relation.ibd             |
 |       7 | ./mysql/help_keyword.ibd              |
 |       8 | ./mysql/time_zone_name.ibd            |
 |       9 | ./mysql/time_zone.ibd                 |
 |      10 | ./mysql/time_zone_transition.ibd      |
 |      11 | ./mysql/time_zone_transition_type.ibd |
 |      12 | ./mysql/time_zone_leap_second.ibd     |
 |      13 | ./mysql/innodb_table_stats.ibd        |
 |      14 | ./mysql/innodb_index_stats.ibd        |
 |      15 | ./mysql/slave_relay_log_info.ibd      |
 |      16 | ./mysql/slave_master_info.ibd         |
 |      17 | ./mysql/slave_worker_info.ibd         |
 |      18 | ./mysql/gtid_executed.ibd             |
 |      19 | ./mysql/server_cost.ibd               |
 |      20 | ./mysql/engine_cost.ibd               |
 |      21 | ./sys/sys_config.ibd                  |
 |      23 | ./test/t1.ibd                         |
 |      26 | /home/user/test/test/t2.ibd           |
 +---------+---------------------------------------+
```

此查询检索`InnoDB`全局临时表空间的`FILE_ID`和`FILE_NAME`。全局临时表空间文件名以`ibtmp`为前缀。

```sql
mysql> SELECT FILE_ID, FILE_NAME FROM INFORMATION_SCHEMA.FILES
       WHERE FILE_NAME LIKE '%ibtmp%';
+---------+-----------+
| FILE_ID | FILE_NAME |
+---------+-----------+
|      22 | ./ibtmp1  |
+---------+-----------+
```

同样，`InnoDB`撤销表空间文件名以`undo`为前缀。以下查询返回`InnoDB`撤销表空间的`FILE_ID`和`FILE_NAME`。

```sql
mysql> SELECT FILE_ID, FILE_NAME FROM INFORMATION_SCHEMA.FILES
       WHERE FILE_NAME LIKE '%undo%';
```
