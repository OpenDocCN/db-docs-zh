# 17.21.4 InnoDB 数据字典操作故障排除

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting-datadict.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting-datadict.html)

表定义信息存储在 InnoDB 数据字典中。如果移动数据文件，字典数据可能变得不一致。

如果数据字典损坏或一致性问题阻止您启动`InnoDB`，请参阅第 17.21.3 节，“强制 InnoDB 恢复”以获取有关手动恢复的信息。

#### 无法打开数据文件

启用`innodb_file_per_table`（默认情况下），如果缺少 file-per-table 表空间文件（`.ibd`文件），则可能在启动时出现以下消息：

```sql
[ERROR] InnoDB: Operating system error number 2 in a file operation.
[ERROR] InnoDB: The error means the system cannot find the path specified.
[ERROR] InnoDB: Cannot open datafile for read-only: './test/t1.ibd' OS error: 71
[Warning] InnoDB: Ignoring tablespace `test/t1` because it could not be opened.
```

要解决这些消息，请发出`DROP TABLE`语句，以从数据字典中删除有关缺失表的数据。

#### 恢复孤立的 file-per-table ibd 文件

该过程描述了如何将孤立的 file-per-table`.ibd`文件恢复到另一个 MySQL 实例。如果系统表空间丢失或无法恢复，并且您想要在新的 MySQL 实例上恢复`.ibd`文件备份，则可以使用此过程。

该过程不支持 general tablespace`.ibd`文件。

该过程假定您只有`.ibd`文件备份，您正在恢复到最初创建孤立的`.ibd`文件的 MySQL 版本，并且`.ibd`文件备份是干净的。有关创建干净备份的信息，请参阅第 17.6.1.4 节，“移动或复制 InnoDB 表”。

第 17.6.1.3 节，“导入 InnoDB 表”中概述的表导入限制适用于此过程。

1.  在新的 MySQL 实例中，在同名数据库中重新创建表。

    ```sql
    mysql> CREATE DATABASE sakila;

    mysql> USE sakila;

    mysql> CREATE TABLE actor (
             actor_id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
             first_name VARCHAR(45) NOT NULL,
             last_name VARCHAR(45) NOT NULL,
             last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
             PRIMARY KEY  (actor_id),
             KEY idx_actor_last_name (last_name)
           )ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
    ```

1.  丢弃新创建表的表空间。

    ```sql
    mysql> ALTER TABLE sakila.actor DISCARD TABLESPACE;
    ```

1.  将孤立的`.ibd`文件从备份目录复制到新的数据库目录。

    ```sql
    $> cp /backup_directory/actor.ibd *path/to/mysql-5.7/data*/sakila/
    ```

1.  确保`.ibd`文件具有必要的文件权限。

1.  导入孤立的`.ibd`文件。会发出警告，指示`InnoDB`正在尝试在没有模式验证的情况下导入文件。

    ```sql
    mysql> ALTER TABLE sakila.actor IMPORT TABLESPACE; SHOW WARNINGS;
    Query OK, 0 rows affected, 1 warning (0.15 sec)

    Warning | 1810 | InnoDB: IO Read error: (2, No such file or directory)
    Error opening './sakila/actor.cfg', will attempt to import
    without schema verification
    ```

1.  查询表以验证`.ibd`文件是否成功恢复。

    ```sql
    mysql> SELECT COUNT(*) FROM sakila.actor;
    +----------+
    | count(*) |
    +----------+
    |      200 |
    +----------+
    ```
