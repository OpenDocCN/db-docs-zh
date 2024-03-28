> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-adding-instances.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-adding-instances.html)

#### 20.2.1.6 向组中添加实例

此时，组中有一个成员，服务器 s1，其中包含一些数据。现在是扩展组的时候了，通过添加之前配置的其他两台服务器。

##### 20.2.1.6.1 添加第二个实例

要添加第二个实例，服务器 s2，首先为其创建配置文件。配置与用于服务器 s1 的配置类似，除了诸如`server_id`之类的内容。以下列出了不同的行。

```sql
[mysqld]

#
# Disable other storage engines
#
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"

#
# Replication configuration parameters
#
server_id=2
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE # Not needed from 8.0.21

#
# Group Replication configuration
#
plugin_load_add='group_replication.so'
group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot=off
group_replication_local_address= "s2:33061"
group_replication_group_seeds= "s1:33061,s2:33061,s3:33061"
group_replication_bootstrap_group= off
```

与为服务器 s1 执行的过程类似，放置选项文件后启动服务器。然后按以下方式配置分布式恢复凭据。这些命令与在设置服务器 s1 时使用的命令相同，因为用户在组内共享。此成员需要在第 20.2.1.3 节，“分布式恢复的用户凭据”中配置相同的复制用户。如果您依赖分布式恢复来在所有成员上配置用户，则当 s2 连接到种子 s1 时，复制用户将被复制或克隆到 s1。如果在配置用户凭据时未启用二进制日志记录，并且不使用远程克隆操作进行状态传输，则必须在 s2 上创建复制用户。在这种情况下，连接到 s2 并发出：

```sql
SET SQL_LOG_BIN=0;
CREATE USER *rpl_user*@'%' IDENTIFIED BY '*password*';
GRANT REPLICATION SLAVE ON *.* TO *rpl_user*@'%';
GRANT CONNECTION_ADMIN ON *.* TO *rpl_user*@'%';
GRANT BACKUP_ADMIN ON *.* TO *rpl_user*@'%';
GRANT GROUP_REPLICATION_STREAM ON *.* TO *rpl_user*@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
```

如果您正在使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句提供用户凭据，请在此之后发出以下语句：

```sql
CHANGE MASTER TO MASTER_USER='*rpl_user*', MASTER_PASSWORD='*password*' \\
	FOR CHANNEL 'group_replication_recovery';

Or from MySQL 8.0.23:
CHANGE REPLICATION SOURCE TO SOURCE_USER='*rpl_user*', SOURCE_PASSWORD='*password*' \\
	FOR CHANNEL 'group_replication_recovery';
```

提示

如果您正在使用缓存的 SHA-2 身份验证插件，这是 MySQL 8 中的默认设置，请参阅第 20.6.3.1.1 节，“具有缓存 SHA-2 身份验证插件的复制用户”。

如有必要，安装组复制插件，请参阅第 20.2.1.4 节，“启动组复制”。

启动组复制，s2 开始加入组的过程。

```sql
mysql> START GROUP_REPLICATION;
```

或者，如果您正在为`START GROUP_REPLICATION`语句提供分布式恢复的用户凭据（从 MySQL 8.0.21 开始）：

```sql
mysql> START GROUP_REPLICATION USER='*rpl_user*', PASSWORD='*password*';
```

与在 s1 上执行的先前步骤不同，这里的区别在于您*不*需要引导组，因为组已经存在。换句话说，在 s2 上 `group_replication_bootstrap_group` 设置为`OFF`，并且在启动 Group Replication 之前不需要发出`SET GLOBAL group_replication_bootstrap_group=ON;`，因为组已经由服务器 s1 创建和引导。此时，服务器 s2 只需要添加到已经存在的组中。

提示

当 Group Replication 成功启动并且服务器加入组时，它会检查 `super_read_only` 变量。通过在成员的配置文件中将 `super_read_only` 设置为 ON，您可以确保由于任何原因在启动 Group Replication 时失败的服务器不接受事务。如果服务器应该作为读写实例加入组，例如作为单主组中的主服务器或作为多主组的成员，当 `super_read_only` 变量设置为 ON 时，加入组时它会被设置为 OFF。

再次检查 `performance_schema.replication_group_members` 表，显示组中现在有两个`ONLINE`服务器。

```sql
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 |   s1        |        3306 | ONLINE       | PRIMARY     | 8.0.27         | XCom                       |
| group_replication_applier | ac39f1e6-6dfa-11e6-a69d-00212844f856 |   s2        |        3306 | ONLINE       | SECONDARY   | 8.0.27         | XCom                       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
```

当 s2 尝试加入组时，第 20.5.4 节，“分布式恢复” 确保 s2 应用了与 s1 应用的相同事务。一旦此过程完成，s2 可以作为成员加入组，并在此时标记为`ONLINE`。换句话说，它必须已经自动赶上了服务器 s1。一旦 s2 处于`ONLINE`状态，它就开始与组处理事务。验证 s2 是否已经与服务器 s1 同步如下。

```sql
mysql> SHOW DATABASES LIKE 'test';
+-----------------+
| Database (test) |
+-----------------+
| test            |
+-----------------+

mysql> SELECT * FROM test.t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+

mysql> SHOW BINLOG EVENTS;
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         2 |         123 | Server ver: 8.0.36-log, Binlog ver: 4                              |
| binlog.000001 |  123 | Previous_gtids |         2 |         150 |                                                                    |
| binlog.000001 |  150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 |  211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 |  270 | View_change    |         1 |         369 | view_id=14724832985483517:1                                        |
| binlog.000001 |  369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 |  434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 |  495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 |  585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 |  646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 |  770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 |  831 | Query          |         1 |         890 | BEGIN                                                              |
| binlog.000001 |  890 | Table_map      |         1 |         933 | table_id: 108 (test.t1)                                            |
| binlog.000001 |  933 | Write_rows     |         1 |         975 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 |  975 | Xid            |         1 |        1002 | COMMIT /* xid=30 */                                                |
| binlog.000001 | 1002 | Gtid           |         1 |        1063 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:5'  |
| binlog.000001 | 1063 | Query          |         1 |        1122 | BEGIN                                                              |
| binlog.000001 | 1122 | View_change    |         1 |        1261 | view_id=14724832985483517:2                                        |
| binlog.000001 | 1261 | Query          |         1 |        1326 | COMMIT                                                             |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
```

如上所示，第二个服务器已添加到组中，并且已自动复制了来自服务器 s1 的更改。换句话说，应用在 s1 上直到 s2 加入组的时间点的事务已经复制到 s2。

##### 20.2.1.6.2 添加额外实例

将额外的实例添加到组中本质上与添加第二个服务器的步骤相同，只是配置必须像对服务器 s2 进行配置一样进行更改。总结所需的命令如下：

1.  创建配置文件。

    ```sql
    [mysqld]

    #
    # Disable other storage engines
    #
    disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"

    #
    # Replication configuration parameters
    #
    server_id=3
    gtid_mode=ON
    enforce_gtid_consistency=ON
    binlog_checksum=NONE # Not needed from 8.0.21

    #
    # Group Replication configuration
    #
    plugin_load_add='group_replication.so'
    group_replication_group_name="aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
    group_replication_start_on_boot=off
    group_replication_local_address= "s3:33061"
    group_replication_group_seeds= "s1:33061,s2:33061,s3:33061"
    group_replication_bootstrap_group= off
    ```

1.  启动服务器并连接到它。为分布式恢复创建复制用户。

    ```sql
    SET SQL_LOG_BIN=0;
    CREATE USER *rpl_user*@'%' IDENTIFIED BY '*password*';
    GRANT REPLICATION SLAVE ON *.* TO *rpl_user*@'%';
    GRANT CONNECTION_ADMIN ON *.* TO *rpl_user*@'%';
    GRANT BACKUP_ADMIN ON *.* TO *rpl_user*@'%';
    GRANT GROUP_REPLICATION_STREAM ON *.* TO *rpl_user*@'%';
    FLUSH PRIVILEGES;
    SET SQL_LOG_BIN=1;
    ```

    如果您正在使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句提供用户凭据，请在此之后发出以下语句：

    ```sql
    CHANGE MASTER TO MASTER_USER='*rpl_user*', MASTER_PASSWORD='*password*' \\
    	FOR CHANNEL 'group_replication_recovery';

    Or from MySQL 8.0.23:
    CHANGE REPLICATION SOURCE TO SOURCE_USER='*rpl_user*', SOURCE_PASSWORD='*password*' \\
    	FOR CHANNEL 'group_replication_recovery';
    ```

1.  如有必要，安装组复制插件。

    ```sql
    INSTALL PLUGIN group_replication SONAME 'group_replication.so';
    ```

1.  启动组复制。

    ```sql
    mysql> START GROUP_REPLICATION;
    ```

    或者，如果您正在为分布式恢复提供用户凭据，可以在`START GROUP_REPLICATION`语句中执行此操作（从 MySQL 8.0.21 开始）：

    ```sql
    mysql> START GROUP_REPLICATION USER='*rpl_user*', PASSWORD='*password*';
    ```

此时服务器 s3 已经启动并运行，已加入组并赶上了组内其他服务器的进度。再次查看`performance_schema.replication_group_members`表可以确认这一点。

```sql
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 395409e1-6dfa-11e6-970b-00212844f856 |   s1        |        3306 | ONLINE       | PRIMARY     | 8.0.27         | XCom                       |
| group_replication_applier | 7eb217ff-6df3-11e6-966c-00212844f856 |   s3        |        3306 | ONLINE       | SECONDARY   | 8.0.27         | XCom                       |
| group_replication_applier | ac39f1e6-6dfa-11e6-a69d-00212844f856 |   s2        |        3306 | ONLINE       | SECONDARY   | 8.0.27         | XCom                       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
```

在服务器 s2 或服务器 s1 上发出相同的查询会得到相同的结果。此外，您可以验证服务器 s3 已经赶上：

```sql
mysql> SHOW DATABASES LIKE 'test';
+-----------------+
| Database (test) |
+-----------------+
| test            |
+-----------------+

mysql> SELECT * FROM test.t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+

mysql> SHOW BINLOG EVENTS;
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |    4 | Format_desc    |         3 |         123 | Server ver: 8.0.36-log, Binlog ver: 4                              |
| binlog.000001 |  123 | Previous_gtids |         3 |         150 |                                                                    |
| binlog.000001 |  150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 |  211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 |  270 | View_change    |         1 |         369 | view_id=14724832985483517:1                                        |
| binlog.000001 |  369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 |  434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 |  495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 |  585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 |  646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 |  770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 |  831 | Query          |         1 |         890 | BEGIN                                                              |
| binlog.000001 |  890 | Table_map      |         1 |         933 | table_id: 108 (test.t1)                                            |
| binlog.000001 |  933 | Write_rows     |         1 |         975 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 |  975 | Xid            |         1 |        1002 | COMMIT /* xid=29 */                                                |
| binlog.000001 | 1002 | Gtid           |         1 |        1063 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:5'  |
| binlog.000001 | 1063 | Query          |         1 |        1122 | BEGIN                                                              |
| binlog.000001 | 1122 | View_change    |         1 |        1261 | view_id=14724832985483517:2                                        |
| binlog.000001 | 1261 | Query          |         1 |        1326 | COMMIT                                                             |
| binlog.000001 | 1326 | Gtid           |         1 |        1387 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:6'  |
| binlog.000001 | 1387 | Query          |         1 |        1446 | BEGIN                                                              |
| binlog.000001 | 1446 | View_change    |         1 |        1585 | view_id=14724832985483517:3                                        |
| binlog.000001 | 1585 | Query          |         1 |        1650 | COMMIT                                                             |
+---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
```
