# 20.5.6 使用 MySQL Enterprise Backup 与组复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-enterprise-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-enterprise-backup.html)

MySQL Enterprise Backup 是 MySQL Server 的商业许可备份实用程序，可与[MySQL Enterprise Edition](https://www.mysql.com/products/enterprise/)一起使用。本节解释了如何使用 MySQL Enterprise Backup 备份并随后恢复组复制成员。相同的技术可以用于快速向组中添加新成员。

#### 使用 MySQL Enterprise Backup 备份组复制成员

备份组复制成员类似于备份独立的 MySQL 实例。以下说明假定您已经熟悉如何使用 MySQL Enterprise Backup 执行备份；如果不是这种情况，请查看 MySQL Enterprise Backup 8.0 用户指南，特别是备份数据库服务器。还请注意授予 MySQL 特权给备份管理员和使用 MySQL Enterprise Backup 与组复制中描述的要求。

考虑以下具有三个成员`s1`、`s2`和`s3`的组，运行在具有相同名称的主机上：

```sql
mysql> SELECT member_host, member_port, member_state FROM performance_schema.replication_group_members;
+-------------+-------------+--------------+
| member_host | member_port | member_state |
+-------------+-------------+--------------+
| s1          |        3306 | ONLINE       |
| s2          |        3306 | ONLINE       |
| s3          |        3306 | ONLINE       |
+-------------+-------------+--------------+
```

使用 MySQL Enterprise Backup，在其主机上发出以下命令，例如，创建`s2`的备份：

```sql
s2> mysqlbackup --defaults-file=/etc/my.cnf --backup-image=/backups/my.mbi_`date +%d%m_%H%M` \
		      --backup-dir=/backups/backup_`date +%d%m_%H%M` --user=root -p \
--host=127.0.0.1 backup-to-image
```

注意

+   *对于 MySQL Enterprise Backup 8.0.18 及更早版本，*如果系统变量`sql_require_primary_key`设置为`ON`，MySQL Enterprise Backup 无法在服务器上记录备份进度。这是因为服务器上的`backup_progress`表是一个 CSV 表，不支持主键。在这种情况下，**mysqlbackup**在备份操作期间会发出以下警告：

    ```sql
    181011 11:17:06 MAIN WARNING: MySQL query 'CREATE TABLE IF NOT EXISTS
    mysql.backup_progress( `backup_id` BIGINT NOT NULL, `tool_name` VARCHAR(4096)
    NOT NULL, `error_code` INT NOT NULL, `error_message` VARCHAR(4096) NOT NULL,
    `current_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP               ON
    UPDATE CURRENT_TIMESTAMP,`current_state` VARCHAR(200) NOT NULL ) ENGINE=CSV
    DEFAULT CHARSET=utf8mb3 COLLATE=utf8mb3_bin': 3750, Unable to create a table
    without PK, when system variable 'sql_require_primary_key' is set. Add a PK
    to the table or unset this variable to avoid this message. Note that tables
    without PK can cause performance problems in row-based replication, so please
    consult your DBA before changing this setting.
    181011 11:17:06 MAIN WARNING: This backup operation's progress info cannot be
    logged.
    ```

    这并不会阻止**mysqlbackup**完成备份。

+   *对于 MySQL Enterprise Backup 8.0.20 及更早版本*，当备份次要成员时，由于 MySQL Enterprise Backup 无法将备份状态和元数据写入只读服务器实例，可能会在备份操作期间发出类似以下警告：

    ```sql
    181113 21:31:08 MAIN WARNING: This backup operation cannot write to backup
    progress. The MySQL server is running with the --super-read-only option.
    ```

    您可以通过在备份命令中使用`--no-history-logging`选项来避免警告。对于 MySQL Enterprise Backup 8.0.21 及更高版本，这不是问题—请参阅使用 MySQL Enterprise Backup 与组复制获取详细信息。

#### 恢复失败的成员

假设其中一个成员（以下示例���的`s3`）无法协调。可以使用组成员`s2`的最新备份来恢复`s3`。以下是执行恢复的步骤：

1.  *将 s2 的备份复制到 s3 的主机上。*复制备份的确切方式取决于您可用的操作系统和工具。在本例中，我们假设主机都是 Linux 服务器，并使用 SCP 在它们之间复制文件：

    ```sql
    s2/backups> scp my.mbi_2206_1429 s3:/backups
    ```

1.  *恢复备份。*连接到目标主机（在本例中为`s3`的主机），并使用 MySQL Enterprise Backup 恢复备份。以下是步骤：

    1.  停止受损服务器，如果它仍在运行。例如，在使用 systemd 的 Linux 发行版上：

        ```sql
        s3> systemctl stop mysqld
        ```

    1.  将受损服务器的数据目录中的两个配置文件`auto.cnf`和`mysqld-auto.cnf`（如果存在）复制到数据目录之外的安全位置以保存服务器的 UUID 和 Section 7.1.9.3，“持久化系统变量”（如果使用），这些在下面的步骤中是必需的。

    1.  删除`s3`的数据目录中的所有内容。例如：

        ```sql
        s3> rm -rf /var/lib/mysql/*
        ```

        如果系统变量`innodb_data_home_dir`，`innodb_log_group_home_dir`，和`innodb_undo_directory`指向除数据目录之外的任何目录，则它们也应该被清空；否则，恢复操作将失败。

    1.  将`s2`的备份恢复到`s3`的主机上：

        ```sql
        s3> mysqlbackup --defaults-file=/etc/my.cnf \
          --datadir=/var/lib/mysql \
          --backup-image=/backups/my.mbi_2206_1429  \
        --backup-dir=/tmp/restore_`date +%d%m_%H%M` copy-back-and-apply-log
        ```

        注意

        上述命令假定`s2`和`s3`上的二进制日志和中继日志具有相同的基本名称，并且位于两个服务器上的相同位置。如果不满足这些条件，您应该使用`--log-bin`和`--relay-log`选项将二进制日志和中继日志恢复到`s3`上的原始文件路径。例如，如果您知道在`s3`上二进制日志的基本名称是`s3-bin`，中继日志的基本名称是`s3-relay-bin`，则您的恢复命令应如下所示：

        ```sql
        mysqlbackup --defaults-file=/etc/my.cnf \
          --datadir=/var/lib/mysql \
          --backup-image=/backups/my.mbi_2206_1429  \
          --log-bin=s3-bin --relay-log=s3-relay-bin \
          --backup-dir=/tmp/restore_`date +%d%m_%H%M` copy-back-and-apply-log
        ```

        能够将二进制日志和中继日志恢复到正确的文件路径会使恢复过程更容易；如果由于某种原因这是不可能的，请参阅重建失败的成员以作为新成员重新加入。

1.  *恢复 s3 的`auto.cnf`文件。*为了重新加入复制组，恢复的成员*必须*具有加入组之前使用的相同的`server_uuid`。通过将在步骤 2 中保留的`auto.cnf`文件复制到恢复成员的数据目录中来提供旧的服务器 UUID。

    注意

    如果您无法通过恢复其旧的`auto.cnf`文件向恢复的成员提供失败成员的原始`server_uuid`，则必须让恢复的成员作为新成员加入组；请参见下面的重建失败的成员以重新加入作为新成员中的说明。

1.  *为`s3`恢复`mysqld-auto.cnf`文件（仅在`s3`使用持久系统变量时需要）。* 必须提供用于配置失败成员的 Section 7.1.9.3，“持久系统变量”的设置，这些设置可以在失败服务器的`mysqld-auto.cnf`文件中找到，您应该在上述第 2 步中保留该文件。将文件恢复到恢复服务器的数据目录中。请参阅恢复持久系统变量以了解如果没有文件副本该怎么办。

1.  *启动恢复服务器。* 例如，在使用 systemd 的 Linux 发行版上：

    ```sql
    systemctl start mysqld
    ```

    注意

    如果您正在恢复的服务器是主要成员，请在启动恢复服务器之前执行恢复主要成员中描述的步骤。

1.  *重新启动 Group Replication。* 连接到重新启动的`s3`，例如，使用**mysql**客户端，并执行以下命令：

    ```sql
    mysql> START GROUP_REPLICATION;
    ```

    在恢复的实例可以成为组的在线成员之前，需要应用在备份之后发生的任何事务；这是通过使用 Group Replication 的分布式恢复机制实现的，该过程在发出 START GROUP_REPLICATION 语句后开始。要检查恢复实例的成员状态，请执行：

    ```sql
    mysql> SELECT member_host, member_port, member_state FROM performance_schema.replication_group_members;
    +-------------+-------------+--------------+
    | member_host | member_port | member_state |
    +-------------+-------------+--------------+
    | s1          |        3306 | ONLINE       |
    | s2          |        3306 | ONLINE       |
    | s3          |        3306 | RECOVERING   |
    +-------------+-------------+--------------+
    ```

    这表明`s3`正在应用事务以赶上组的进度。一旦它赶上了组的其他成员，它的`member_state`就会变为`ONLINE`：

    ```sql
    mysql> SELECT member_host, member_port, member_state FROM performance_schema.replication_group_members;
    +-------------+-------------+--------------+
    | member_host | member_port | member_state |
    +-------------+-------------+--------------+
    | s1          |        3306 | ONLINE       |
    | s2          |        3306 | ONLINE       |
    | s3          |        3306 | ONLINE       |
    +-------------+-------------+--------------+
    ```

    注意

    如果您正在恢复的服务器是主要成员，一旦它与组同步并变为`ONLINE`，请执行恢复主要成员末尾描述的步骤，以恢复您在启动服务器之前所做的配置更改。

该成员现在已经完全从备份中恢复，并作为组的常规成员运行。

#### 重建失败的成员以重新加入作为新成员

有时，恢复失败成员中概述的步骤无法执行，因为例如二进制日志或中继日志已损坏，或者在备份中丢失。在这种情况下，使用备份重建成员，然后将其作为新成员添加到组中。在以下步骤中，我们假设重建的成员命名为`s3`，与失败的成员相同，并且在与`s3`相同的主机上运行：

1.  *将`s2`的备份复制到`s3`的主机上*。复制备份的确切方式取决于您所使用的操作系统和可用工具。在本示例中，我们假设主机都是 Linux 服务器，并使用 SCP 在它们之间复制文件：

    ```sql
    s2/backups> scp my.mbi_2206_1429 s3:/backups
    ```

1.  *恢复备份*。连接到目标主机（在本例中为`s3`的主机），并使用 MySQL Enterprise Backup 恢复备份。以下是步骤：

    1.  停止受损的服务器，如果它仍在运行。例如，在使用 systemd 的 Linux 发行版上：

        ```sql
        s3> systemctl stop mysqld
        ```

    1.  保留配置文件`mysqld-auto.cnf`，如果在受损服务器的数据目录中找到，则将其复制到数据目录之外的安全位置。这是为了保留服务器的第 7.1.9.3 节，“持久化系统变量”，稍后会用到。

    1.  删除`s3`的数据目录中的所有内容。例如：

        ```sql
        s3> rm -rf /var/lib/mysql/*
        ```

        如果系统变量`innodb_data_home_dir`、`innodb_log_group_home_dir`和`innodb_undo_directory`指向除数据目录以外的任何目录，则这些目录也应该清空；否则，恢复操作将失败。

    1.  将`s2`的备份恢复到`s3`的主机上。通过这种方法，我们正在将`s3`重新构建为一个新成员，因此我们不需要或不想使用备份中的旧二进制日志和中继日志；因此，如果这些日志已包含在您的备份中，请使用`--skip-binlog`和`--skip-relaylog`选项排除它们：

        ```sql
        s3> mysqlbackup --defaults-file=/etc/my.cnf \
          --datadir=/var/lib/mysql \
          --backup-image=/backups/my.mbi_2206_1429  \
          --backup-dir=/tmp/restore_`date +%d%m_%H%M` \
          --skip-binlog --skip-relaylog \
        copy-back-and-apply-log
        ```

        注意

        如果您在备份中有健康的二进制日志和中继日志，可以毫无问题地将其传输到目标主机上，建议您按照上述恢复失败成员中描述的更简单的过程进行操作。

1.  *为`s3`恢复`mysqld-auto.cnf`文件（仅在`s3`使用持久系统变量时需要）。* 必须提供用于配置失败成员的 Section 7.1.9.3, “Persisted System Variables”的设置，这些设置可以在失败服务器的`mysqld-auto.cnf`文件中找到，您应该在上述第 2 步中保留该文件。将文件恢复到恢复服务器的数据目录中。如果您没有文件副本，则请参阅恢复持久系统变量。

    注意

    不要将损坏服务器的`auto.cnf`文件恢复到新成员的数据目录中——当重建的`s3`作为新成员加入组时，它将被分配一个新的服务器 UUID。

1.  *启动恢复的服务器。* 例如，在使用 systemd 的 Linux 发行版上：

    ```sql
    systemctl start mysqld
    ```

    注意

    如果您正在恢复的服务器是主要成员，请在*启动恢复的服务器*之前执行恢复主要成员中描述的步骤。

1.  *重新配置恢复的成员以加入组复制。* 使用**mysql**客户端连接到恢复的服务器，并使用以下命令重置源和副本信息：

    ```sql
    mysql> RESET MASTER;
    ```

    ```sql
    mysql> RESET SLAVE ALL;
    Or from MySQL 8.0.22:
    mysql> RESET REPLICA ALL;
    ```

    为了使恢复的服务器能够使用组复制的内置机制进行分布式恢复，配置服务器的`gtid_executed`变量。为此，请使用备份的`s2`中包含的`backup_gtid_executed.sql`文件，该文件通常在恢复成员的数据目录下恢复。禁用二进制日志记录，使用`backup_gtid_executed.sql`文件配置`gtid_executed`，然后通过您的**mysql**客户端发出以下语句重新启用二进制日志记录：

    ```sql
    mysql> SET SQL_LOG_BIN=OFF;
    mysql> SOURCE *datadir*/backup_gtid_executed.sql
    mysql> SET SQL_LOG_BIN=ON;
    ```

    然后，在成员上配置组复制用户凭据：

    ```sql
    mysql> CHANGE MASTER TO MASTER_USER='*rpl_user*', MASTER_PASSWORD='*password*' /
    		FOR CHANNEL 'group_replication_recovery';

    Or from MySQL 8.0.23:
    mysql> CHANGE REPLICATION SOURCE TO SOURCE_USER='*rpl_user*', SOURCE_PASSWORD='*password*' /
    		FOR CHANNEL 'group_replication_recovery';
    ```

1.  *重新启动组复制。* 使用您的**mysql**客户端向恢复的服务器发出以下命令：

    ```sql
    mysql> START GROUP_REPLICATION;
    ```

    在恢复的实例可以成为组的在线成员之前，需要将备份后组中发生的任何事务应用到实例上；这是通过使用 Group Replication 的分布式恢复机制实现的，该过程在发出 START GROUP_REPLICATION 语句后开始。要检查恢复实例的成员状态，请执行：

    ```sql
    mysql> SELECT member_host, member_port, member_state FROM performance_schema.replication_group_members;
    +-------------+-------------+--------------+
    | member_host | member_port | member_state |
    +-------------+-------------+--------------+
    | s3          |        3306 | RECOVERING   |
    | s2          |        3306 | ONLINE       |
    | s1          |        3306 | ONLINE       |
    +-------------+-------------+--------------+
    ```

    这表明`s3`正在应用事务以赶上组的进度。一旦它赶上了组的其他成员，其`member_state`将变为`ONLINE`：

    ```sql
    mysql> SELECT member_host, member_port, member_state FROM performance_schema.replication_group_members;
    +-------------+-------------+--------------+
    | member_host | member_port | member_state |
    +-------------+-------------+--------------+
    | s3          |        3306 | ONLINE       |
    | s2          |        3306 | ONLINE       |
    | s1          |        3306 | ONLINE       |
    +-------------+-------------+--------------+
    ```

    注意

    如果您正在恢复的服务器是主要成员，在它与组同步并变为`ONLINE`后，执行恢复主要成员末尾描述的步骤，以恢复在启动之前对服务器所做的配置更改。

该成员现在已经作为新成员恢复到组中。

**恢复持久化系统变量。** **mysqlbackup**不支持备份或保留 Section 7.1.9.3, “Persisted System Variables” ——文件`mysqld-auto.cnf`不包含在备份中。要使用其持久化变量设置启动恢复的成员，您需要执行以下操作之一：

+   保留从损坏服务器复制的`mysqld-auto.cnf`文件，并将其复制到恢复服务器的数据目录。

+   从组的另一个成员复制`mysqld-auto.cnf`文件到恢复服务器的数据目录，如果该成员具有与损坏成员相同的持久化系统变量设置。

+   在启动恢复服务器并重新启动 Group Replication 之前，通过**mysql**客户端手动将所有系统变量设置为其持久化值。

**恢复主要成员。** 如果恢复的成员是组中的主要成员，则在 Group Replication 分布式恢复过程中必须注意防止对恢复数据库的写入。根据客户端访问组的方式，存在在恢复成员在网络上可访问之前执行 DML 语句的可能性，而在成员完成赶上其离线期间错过的活动之前。为了避免这种情况，在*启动恢复服务器*之前，在服务器选项文件中配置以下系统变量：

```sql
group_replication_start_on_boot=OFF
super_read_only=ON
event_scheduler=OFF
```

这些设置确保成员在启动时变为只读，并且在成员在分布式恢复过程中与组进行同步期间关闭事件调度程序。客户端还必须配置足够的错误处理，因为在恢复的成员上在此期间暂时阻止执行 DML 操作。一旦恢复过程完全完成，并且恢复的成员与组的其余部分同步，撤销这些更改；重新启动事件调度程序：

```sql
mysql> SET global event_scheduler=ON;
```

编辑成员选项文件中的以下系统变量，以便为下一次启动正确配置事物：

```sql
group_replication_start_on_boot=ON
super_read_only=OFF
event_scheduler=ON
```
