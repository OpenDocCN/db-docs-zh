> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpuinfo.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpuinfo.html)

#### 25.6.16.17 `ndbinfo` `cpuinfo`表

`cpuinfo`表提供了关于给定数据节点执行的 CPU 的信息。

`cpuinfo`表包含以下列：

+   `node_id`

    节点 ID

+   `cpu_no`

    CPU ID

+   `cpu_online`

    如果 CPU 在线，则为 1，否则为 0

+   `core_id`

    CPU 核心 ID

+   `socket_id`

    CPU 插槽 ID

##### 注意事项

`cpuinfo`表在所有由`NDB`支持的操作系统上都可用，除了 MacOS 和 FreeBSD。

该表在 NDB 8.0.23 中添加。
