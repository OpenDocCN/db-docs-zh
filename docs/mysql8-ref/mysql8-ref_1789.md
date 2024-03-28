> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-hwinfo.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-hwinfo.html)

#### 25.6.16.38 ndbinfo hwinfo 表

`hwinfo`表提供了关于给定数据节点所在硬件的信息。

`hwinfo`表包含以下列：

+   `node_id`

    节点 ID

+   `cpu_cnt_max`

    此主机上的处理器数量

+   `cpu_cnt`

    此节点可用的处理器数量

+   `num_cpu_cores`

    此主机上的 CPU 核心数

+   `num_cpu_sockets`

    此主机上的 CPU 插槽数量

+   `HW_memory_size`

    此主机上可用的内存量

+   `model_name`

    CPU 型号名称

##### 注意事项

`hwinfo`表在所有由`NDB`支持的操作系统上都可用。

此表在 NDB 8.0.23 中添加。
