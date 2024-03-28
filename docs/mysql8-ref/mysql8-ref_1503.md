# 20.4.3 The replication_group_members Table

> [`dev.mysql.com/doc/refman/8.0/en/group-replication-replication-group-members.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-replication-group-members.html)

`performance_schema.replication_group_members`表用于监视组成员的不同服务器实例的状态。表中的信息在视图更改时更新，例如当新成员加入时动态更改组配置时。在那时，服务器交换一些元数据以同步自己并继续共同合作。信息在所有作为复制组成员的服务器实例之间共享，因此可以从任何成员查询所有组成员的信息。该表可用于获取复制组状态的高级视图，例如通过发出：

```sql
SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | d391e9ee-2691-11ec-bf61-00059a3c7a00 | example1    |        4410 | ONLINE       | PRIMARY     | 8.0.27         | XCom                       |
| group_replication_applier | e059ce5c-2691-11ec-8632-00059a3c7a00 | example2    |        4420 | ONLINE       | SECONDARY   | 8.0.27         | XCom                       |
| group_replication_applier | ecd9ad06-2691-11ec-91c7-00059a3c7a00 | example3    |        4430 | ONLINE       | SECONDARY   | 8.0.27         | XCom                       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
3 rows in set (0.0007 sec)
```

根据这个结果，我们可以看到该组由三个成员组成。表中显示了每个成员的`server_uuid`，以及成员的主机名和端口号，客户端用于连接的信息。`MEMBER_STATE`列显示了第 20.4.2 节，“组复制服务器状态”中的一个状态，在这种情况下显示该组的所有三个成员都是`ONLINE`，`MEMBER_ROLE`列显示有两个从属节点和一个主节点。因此，该组必须在单主模式下运行。`MEMBER_VERSION`列在升级组并合并运行不同 MySQL 版本的成员时可能会有用。`MEMBER_COMMUNICATION_STACK`列显示了组使用的通信堆栈。

关于`MEMBER_HOST`值及其对分布式恢复过程的影响的更多信息，请参见第 20.2.1.3 节，“用于分布式恢复的用户凭据”。
