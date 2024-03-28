> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-members-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-members-table.html)

#### 29.12.11.11 The replication_group_members Table

此表显示复制组成员的网络和状态信息。显示的网络地址是用于连接客户端到组的地址，并且不应与由[`group_replication_local_address`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-system-variables.html#sysvar_group_replication_local_address)指定的成员内部组通信地址混淆。

[`replication_group_members`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-members-table.html)表具有以下列：

+   `CHANNEL_NAME`

    Group Replication 通道的名称。

+   `MEMBER_ID`

    成员服务器 UUID。对于组中的每个成员，此值不同。这也作为一个键，因为对于每个成员都是唯一的。

+   `MEMBER_HOST`

    此成员的网络地址（主机名或 IP 地址）。从成员的[`hostname`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_hostname)变量中检索。这是客户端连接的地址，不同于用于内部组通信的 group_replication_local_address。

+   `MEMBER_PORT`

    服务器正在侦听的端口。从成员的[`port`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_port)变量中检索。

+   `MEMBER_STATE`

    此成员的当前状态；可以是以下任何一种：

    +   `ONLINE`: 成员处于完全运作状态。

    +   `RECOVERING`: 服务器已加入正在检索数据的组。

    +   `OFFLINE`: 安装了组复制插件，但尚未启动。

    +   `ERROR`: 成员在应用事务或恢复阶段遇到错误，并且不参与组的事务。

    +   `UNREACHABLE`: 失败检测过程怀疑无法联系到此成员，因为组消息已超时。

    参见[Section 20.4.2, “Group Replication Server States”](https://dev.mysql.com/doc/refman/8.0/en/group-replication-server-states.html)。

+   `MEMBER_ROLE`

    成员在组中的角色，可以是`PRIMARY`或`SECONDARY`。

+   `MEMBER_VERSION`

    成员的 MySQL 版本。

+   `MEMBER_COMMUNICATION_STACK`

    用于组的通信堆栈，可以是`XCOM`通信堆栈或`MYSQL`通信堆栈。

    此列在 MySQL 8.0.27 中添加。

[`replication_group_members`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-members-table.html)表没有索引。

[`TRUNCATE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html)语句不允许用于[`replication_group_members`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-replication-group-members-table.html)表。
