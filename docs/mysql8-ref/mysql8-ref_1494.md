> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-bootstrap.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-bootstrap.html)

#### 20.2.1.5 引导组

首次启动组的过程称为引导。您使用`group_replication_bootstrap_group`系统变量来引导一个组。引导应该只由单个服务器执行，即启动组的服务器，并且只执行一次。这就是为什么`group_replication_bootstrap_group`选项的值未存储在实例的选项文件中。如果它保存在选项文件中，在重新启动时，服务器会自动用相同名称引导第二个组。这将导致具有相同名称的两个不同组。相同的推理适用于设置为`ON`时停止和重新启动插件。因此，为了安全地引导组，请连接到 s1 并执行以下语句：

```sql
mysql> SET GLOBAL group_replication_bootstrap_group=ON;
mysql> START GROUP_REPLICATION;
mysql> SET GLOBAL group_replication_bootstrap_group=OFF;
```

或者，如果您正在为分布式恢复提供用户凭据`START GROUP_REPLICATION`语句（从 MySQL 8.0.21 开始），请执行以下语句：

```sql
mysql> SET GLOBAL group_replication_bootstrap_group=ON;
mysql> START GROUP_REPLICATION USER='*rpl_user*', PASSWORD='*password*';
mysql> SET GLOBAL group_replication_bootstrap_group=OFF;
```

一旦`START GROUP_REPLICATION`语句返回，组已经启动。您可以检查组现在是否已创建，并且其中有一个成员：

```sql
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+---------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE  | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-------------+-------------+---------------+-------------+----------------+----------------------------+
| group_replication_applier | ce9be252-2b71-11e6-b8f4-00212844f856 |   s1        |       3306  | ONLINE        |             |                | XCom                       |
+---------------------------+--------------------------------------+-------------+-------------+---------------+-------------+----------------+----------------------------+
1 row in set (0.0108 sec)
```

这个表中的信息确认了组中有一个具有唯一标识符`ce9be252-2b71-11e6-b8f4-00212844f856`的成员，它处于`ONLINE`状态，并在端口`3306`上的`s1`上监听客户端连接。

用于演示服务器确实处于组中并且能够处理负载的目的，创建一个表并向其中添加一些内容。

```sql
mysql> CREATE DATABASE test;
mysql> USE test;
mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL);
mysql> INSERT INTO t1 VALUES (1, 'Luis');
```

检查表`t1`的内容和二进制日志。

```sql
mysql> SELECT * FROM t1;
+----+------+
| c1 | c2   |
+----+------+
|  1 | Luis |
+----+------+

mysql> SHOW BINLOG EVENTS;
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name      | Pos | Event_type     | Server_id | End_log_pos | Info                                                               |
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
| binlog.000001 |   4 | Format_desc    |         1 |         123 | Server ver: 8.0.36-log, Binlog ver: 4                              |
| binlog.000001 | 123 | Previous_gtids |         1 |         150 |                                                                    |
| binlog.000001 | 150 | Gtid           |         1 |         211 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:1'  |
| binlog.000001 | 211 | Query          |         1 |         270 | BEGIN                                                              |
| binlog.000001 | 270 | View_change    |         1 |         369 | view_id=14724817264259180:1                                        |
| binlog.000001 | 369 | Query          |         1 |         434 | COMMIT                                                             |
| binlog.000001 | 434 | Gtid           |         1 |         495 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:2'  |
| binlog.000001 | 495 | Query          |         1 |         585 | CREATE DATABASE test                                               |
| binlog.000001 | 585 | Gtid           |         1 |         646 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:3'  |
| binlog.000001 | 646 | Query          |         1 |         770 | use `test`; CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 TEXT NOT NULL) |
| binlog.000001 | 770 | Gtid           |         1 |         831 | SET @@SESSION.GTID_NEXT= 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:4'  |
| binlog.000001 | 831 | Query          |         1 |         899 | BEGIN                                                              |
| binlog.000001 | 899 | Table_map      |         1 |         942 | table_id: 108 (test.t1)                                            |
| binlog.000001 | 942 | Write_rows     |         1 |         984 | table_id: 108 flags: STMT_END_F                                    |
| binlog.000001 | 984 | Xid            |         1 |        1011 | COMMIT /* xid=38 */                                                |
+---------------+-----+----------------+-----------+-------------+--------------------------------------------------------------------+
```

如上所示，数据库和表对象已创建，并且它们对应的 DDL 语句已写入二进制日志。此外，数据已插入到表中并写入二进制日志，因此可以通过从捐赠者的二进制日志进行状态传输来用于分布式恢复。
