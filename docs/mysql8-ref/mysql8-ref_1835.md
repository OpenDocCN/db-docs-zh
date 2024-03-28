> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-pitr.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-pitr.html)

#### 25.7.9.2 使用 NDB 集群复制进行时点恢复

NDB 集群表的时点恢复，即恢复自某一特定时间点以来所做的数据更改，是在恢复将服务器恢复到备份时的状态的完整备份之后进行的。使用 NDB 集群和 NDB 集群复制进行 NDB 集群表的时点恢复可以通过使用本地`NDB`数据备份（通过在**ndb_mgm**客户端中发出`CREATE BACKUP`命令）和恢复`ndb_binlog_index`表（从使用**mysqldump**制作的转储中）来完成。

要执行 NDB 集群的时点恢复，需要按照以下步骤进行：

1.  使用`START BACKUP`命令在**ndb_mgm**客户端中备份集群中的所有`NDB`数据库（参见第 25.6.8 节，“NDB 集群的在线备份”）。

1.  在恢复集群之前的某个时间点，备份`mysql.ndb_binlog_index`表。最简单的方法可能是使用**mysqldump**来完成此任务。同时备份二进制日志文件。

    根据您的需求，这个备份应该定期更新，甚至可能每小时更新一次。

1.  （*发生灾难性故障或错误*。）

1.  定位最近的备份。

1.  清除数据节点文件系统（使用**ndbd** `--initial` 或 **ndbmtd**") `--initial`）。

    注意

    从 NDB 8.0.21 开始，磁盘数据表空间和日志文件将通过`--initial`删除。以前，��要手动删除这些文件。

1.  使用`DROP TABLE`或`TRUNCATE TABLE`命令处理`mysql.ndb_binlog_index`表。

1.  执行**ndb_restore**，恢复所有数据。在运行**ndb_restore**时，必须包括`--restore-epoch`选项，以便正确填充`ndb_apply_status`表。（有关更多信息，请参见 Section 25.5.23, “ndb_restore — 恢复 NDB 集群备份”.）

1.  从**mysqldump**的输出中恢复`ndb_binlog_index`表，并根据需要从备份中恢复二进制日志文件。

1.  找到最近应用的时代，即`ndb_apply_status`表中的最大`epoch`列值，作为用户变量`@LATEST_EPOCH`（强调）：

    ```sql
    SELECT *@LATEST_EPOCH*:=MAX(epoch)
        FROM mysql.ndb_apply_status;
    ```

1.  找到与`ndb_binlog_index`表中的`@LATEST_EPOCH`对应的最新二进制日志文件（`@FIRST_FILE`）和位置（`Position`列值）：

    ```sql
    SELECT Position, *@FIRST_FILE*:=File
        FROM mysql.ndb_binlog_index
        WHERE epoch > *@LATEST_EPOCH* ORDER BY epoch ASC LIMIT 1;
    ```

1.  使用**mysqlbinlog**，重放给定文件和位置的二进制日志事件，直到故障点。（参见 Section 6.6.9, “mysqlbinlog — 用于处理二进制日志文件的实用程序”.）

另请参阅 Section 9.5, “时间点（增量）恢复”，了解有关二进制日志、复制和增量恢复的更多信息。
