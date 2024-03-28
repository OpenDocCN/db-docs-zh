> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-config-send-buffers.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-config-send-buffers.html)

#### 25.4.3.14 配置 NDB Cluster 发送缓冲区参数

`NDB` 内核采用统一的发送缓冲区，其内存是动态分配的，来自所有传输器共享的池。这意味着发送缓冲区的大小可以根据需要进行调整。统一发送缓冲区的配置可以通过设置以下参数来完成：

+   **TotalSendBufferMemory. ** 此参数可为所有类型的 NDB Cluster 节点设置，即可在 `config.ini` 文件的 `[ndbd]`、`[mgm]` 和 `[api]`（或 `[mysql]`）部分中设置。它表示每个设置了该参数的节点为所有配置的传输器分配的总内存量（以字节为单位）。如果设置，其最小值为 256KB；最大值为 4294967039。

    为了向后兼容现有配置，此参数的默认值为所有配置传输器的最大发送缓冲区大小之和，再加上每个传输器额外的 32KB（一个页面）。最大值取决于传输器类型，如下表所示：

    **表 25.23 具有最大发送缓冲区大小的传输器类型**

    | 传输器 | 最大发送缓冲区大小（字���） |
    | --- | --- |
    | TCP | `SendBufferMemory`（默认值 = 2M） |
    | SHM | 20K |

    这使得现有配置可以在与 NDB Cluster 6.3 及更早版本几乎相同的方式下运行，每个传输器都有相同数量的内存和发送缓冲区空间可用。然而，一个传输器未使用的内存对其他传输器不可用。

+   **OverloadLimit. ** 此参数用于 `config.ini` 文件中的 `[tcp]` 部分，表示在连接被认为过载之前必须存在于发送缓冲区中的未发送数据量（以字节为单位）。当发生这种过载情况时，影响过载连接的事务将因 NDB API 错误 1218（NDB 内核中的发送缓冲区过载）而失败，直到过载状态消失。默认值为 0，此时对于给定连接，有效的过载限制计算为 `SendBufferMemory * 0.8`。此参数的最大值为 4G。

+   **SendBufferMemory. ** 此值表示单个传输器可以使用的内存量的硬限制，该内存量来自由 `TotalSendBufferMemory` 指定的整个池。然而，所有配置的传输器的 `SendBufferMemory` 总和可能大于为给定节点设置的 `TotalSendBufferMemory`。当许多节点在使用时，这是一种节省内存的方法，只要所有传输器在同一时间从未同时需要最大内存量。

你可以使用`ndbinfo.transporters`表来监控发送缓冲区内存使用情况，并检测可能对性能产生不利影响的减速和过载条件。
