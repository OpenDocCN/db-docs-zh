# 17.15.7 InnoDB INFORMATION_SCHEMA 临时表信息表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-temp-table-info.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-temp-table-info.html)

`INNODB_TEMP_TABLE_INFO`提供有关活动在`InnoDB`实例中的用户创建的`InnoDB`临时表的信息。它不提供有关优化器使用的内部`InnoDB`临时表的信息。

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB_TEMP%';
+---------------------------------------------+
| Tables_in_INFORMATION_SCHEMA (INNODB_TEMP%) |
+---------------------------------------------+
| INNODB_TEMP_TABLE_INFO                      |
+---------------------------------------------+
```

有关表定义，请参见第 28.4.27 节，“INFORMATION_SCHEMA INNODB_TEMP_TABLE_INFO 表”。

**示例 17.12 INNODB_TEMP_TABLE_INFO**

此示例演示了`INNODB_TEMP_TABLE_INFO`表的特性。

1.  创建一个简单的`InnoDB`临时表：

    ```sql
    mysql> CREATE TEMPORARY TABLE t1 (c1 INT PRIMARY KEY) ENGINE=INNODB;
    ```

1.  查询`INNODB_TEMP_TABLE_INFO`以查看临时表元数据。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TEMP_TABLE_INFO\G
    *************************** 1\. row ***************************
                TABLE_ID: 194
                    NAME: #sql7a79_1_0
                  N_COLS: 4
                   SPACE: 182
    ```

    `TABLE_ID`是临时表的唯一标识符。`NAME`列显示临时表的系统生成名称，前缀为“#sql”。列数（`N_COLS`）为 4，而不是 1，因为`InnoDB`始终创建三个隐藏表列（`DB_ROW_ID`、`DB_TRX_ID`和`DB_ROLL_PTR`）。

1.  重启 MySQL 并查询`INNODB_TEMP_TABLE_INFO`。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TEMP_TABLE_INFO\G
    ```

    返回一个空集，因为当服务器关闭时，`INNODB_TEMP_TABLE_INFO`及其数据不会持久保存到磁盘。

1.  创建一个新的临时表。

    ```sql
    mysql> CREATE TEMPORARY TABLE t1 (c1 INT PRIMARY KEY) ENGINE=INNODB;
    ```

1.  查询`INNODB_TEMP_TABLE_INFO`以查看临时表元数据。

    ```sql
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TEMP_TABLE_INFO\G
    *************************** 1\. row ***************************
                TABLE_ID: 196
                    NAME: #sql7b0e_1_0
                  N_COLS: 4
                   SPACE: 184
    ```

    `SPACE` ID 可能不同，因为它在服务器启动时动态生成。
