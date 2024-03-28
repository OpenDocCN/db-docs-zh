> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tcp-definition.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-tcp-definition.html)

#### 25.4.3.10 NDB 集群 TCP/IP 连接

TCP/IP 是 NDB 集群中所有节点之间连接的默认传输机制。通常情况下，不需要定义 TCP/IP 连接；NDB 集群会自动为所有数据节点、管理节点以及 SQL 或 API 节点设置这些连接。

注意

对于此规则的例外情况，请参阅第 25.4.3.11 节，“使用直接连接的 NDB 集群 TCP/IP 连接”。

要覆盖默认连接参数，需要在`config.ini`文件中使用一个或多个`[tcp]`部分定义连接。每个`[tcp]`部分明确定义了两个 NDB 集群节点之间的 TCP/IP 连接，必须至少包含参数`NodeId1`和`NodeId2`，以及任何要覆盖的连接参数。

也可以通过在`[tcp default]`部分中设置这些参数的默认值来更改这些参数的默认值。

重要

`config.ini`文件中的任何`[tcp]`部分应该在所有其他部分之后列出。但是，对于`[tcp default]`部分，不需要这样做。这是 NDB 集群管理服务器读取`config.ini`文件的方式存在的已知问题。

可以在`config.ini`文件的`[tcp]`和`[tcp default]`部分中设置的连接参数列在此处：

+   `AllowUnresolvedHostNames`

    | 版本（或更高版本） | NDB 8.0.22 |
    | --- | --- |
    | 类型或单位 | 布尔值 |
    | 默认值 | false |
    | 范围 | true, false |
    | 添加 | NDB 8.0.22 |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    默认情况下，当管理节点在连接时无法解析主机名时，会导致致命错误。可以通过在全局配置文件（通常命名为`config.ini`）的`[tcp default]`部分中将`AllowUnresolvedHostNames`设置为`true`来覆盖此行为，在这种情况下，无法解析主机名将被视为警告，**ndb_mgmd**启动将继续无中断。

+   `Checksum`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 布尔值 |
    | 默认值 | false |
    | 范围 | true, false |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    此参数默认情况下处于禁用状态。当启用时（设置为 `Y` 或 `1`），所有消息在放入发送缓冲区之前都会计算校验和。此功能确保消息在等待发送缓冲区或通过传输机制时不会损坏。

+   `Group`

    当启用 `ndb_optimized_node_selection` 时，节点的接近程度在某些情况下用于选择要连接的节点。可以通过将此参数设置为较低值来影响接近程度，较低值被解释为“更近”。有关更多信息，请参阅系统变量的描述。

+   `HostName1`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 名称或 IP 地址 |
    | 默认 | [...] |
    | 范围 | ... |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    `HostName1` 和 `HostName2` 参数可用于指定两个节点之间的 TCP 连接所使用的特定网络接口。这些参数的值可以是主机名或 IP 地址。

+   `HostName2`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 名称或 IP 地址 |
    | 默认 | [...] |
    | 范围 | ... |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    `HostName1` 和 `HostName2` 参数可用于指定两个节点之间的 TCP 连接所使用的特定网络接口。这些参数的值可以是主机名或 IP 地址。

+   `NodeId1`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 数字 |
    | 默认 | [none] |
    | 范围 | 1 - 255 |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    要识别两个节点之间的连接，需要在配置文件的 `[tcp]` 部分中提供它们的节点 ID 作为 `NodeId1` 和 `NodeId2` 的值。这些值是每个节点的唯一 `Id` 值，如第 25.4.3.7 节，“在 NDB 集群中定义 SQL 和其他 API 节点”中所述。

+   `NodeId2`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 数�� |
    | 默认 | [none] |
    | 范围 | 1 - 255 |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    要识别两个节点之间的连接，需要在配置文件的 `[tcp]` 部分提供它们的节点 ID 作为 `NodeId1` 和 `NodeId2` 的值。这些是每个节点的相同唯一 `Id` 值，如第 25.4.3.7 节，“在 NDB 集群中定义 SQL 和其他 API 节点”中所述。

+   `NodeIdServer`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 数值 |
    | 默认 | [无] |
    | 范围 | 1 - 63 |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    设置 TCP 连接的服务器端。

+   `OverloadLimit`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字节 |
    | 默认 | 0 |
    | 范围 | 0 - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    当发送缓冲区中有超过这么多未发送字节时，连接被视为过载。

    此参数可用于确定在连接被认为过载之前必须存在的未发送数据量。有关更多信息，请参见第 25.4.3.14 节，“配置 NDB 集群发送缓冲区参数”。

+   `PreferIPVersion`

    | 版本（或更高版本） | NDB 8.0.26 |
    | --- | --- |
    | 类型或单位 | 枚举 |
    | 默认 | 4 |
    | 范围 | 4, 6 |
    | 添加 | NDB 8.0.26 |
    | 重启类型 | **初始系统重启：** 需要完全关闭集群，从备份中擦除和恢复集群文件系统，然后重新启动集群。（NDB 8.0.13） |

    确定 DNS 解析对 IP 版本 4 或版本 6 的偏好。因为 NDB 集群所采用的配置检索机制要求所有连接使用相同的偏好，所以应该在 `config.ini` 全局配置文件的 `[tcp default]` 中设置此参数。

+   `PreSendChecksum`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 布尔值 |
    | 默认 | false |
    | 范围 | true, false |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    如果启用了此参数和`Checksum`，则执行预发送校验和检查，并检查节点之间的所有 TCP 信号是否存在错误。如果未启用`Checksum`，则不起作用。

+   `Proxy`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字符串 |
    | 默认值 | [...] |
    | 范围 | ... |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    为 TCP 连接设置代理。

+   `ReceiveBufferMemory`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字节 |
    | 默认值 | 2M |
    | 范围 | 16K - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    指定从 TCP/IP 套接字接收数据时使用的缓冲区大小。

    此参数的默认值为 2MB。最小可能值为 16KB；理论上的最大值为 4GB。

+   `SendBufferMemory`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | 2M |
    | 范围 | 256K - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    TCP 传输器使用缓冲区在执行发送调用到操作系统之前存储所有消息。当此缓冲区达到 64KB 时，其内容将被发送；当一轮消息已执行时，也会发送。为处理临时过载情况，还可以定义更大的发送缓冲区。

    如果显式设置了此参数，则内存不会专门分配给每个传输器；相反，使用的值表示单个传输器可以使用的内存硬限制（即`TotalSendBufferMemory`总可用内存的一部分）。有关在 NDB Cluster 中配置动态传输器发送缓冲区内存分配的更多信息，请参见 Section 25.4.3.14，“配置 NDB Cluster 发送缓冲区参数”。

    发送缓冲区的默认大小为 2MB，在大多数情况下建议使用此大小。最小大小为 64KB；理论上的最大值为 4GB。

+   `SendSignalId`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 布尔值 |
    | 默认值 | false（调试版本为 true） |
    | 范围 | true, false |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    为了能够追溯分布式消息数据报，有必要标识每个消息。当此参数设置为 `Y` 时，消息 ID 被传输到网络上。此功能在生产构建中默认禁用，在 `-debug` 构建中启用。

+   `TcpBind_INADDR_ANY`

    将此参数设置为 `TRUE` 或 `1` 将绑定 `IP_ADDR_ANY`，以便可以从任何地方进行连接（用于自动生成的连接）。默认值为 `FALSE`（`0`）。

+   `TcpSpinTime`

    | 版本（或更高版本） | NDB 8.0.20 |
    | --- | --- |
    | 类型或单位 | 微秒 |
    | 默认 | 0 |
    | 范围 | 0 - 2000 |
    | 添加 | NDB 8.0.20 |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    控制 TCP 传输器的自旋；若要启用，请设置为非零值。这适用于数据节点以及连接的管理或 SQL 节点的两侧。

+   `TCP_MAXSEG_SIZE`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认 | 0 |
    | 范围 | 0 - 2G |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    确定在 TCP 传输器初始化期间设置的内存大小。对于大多数常见用例，建议使用默认值。

+   `TCP_RCV_BUF_SIZE`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认 | 0 |
    | 范围 | 0 - 2G |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    确定在 TCP 传输器初始化期间设置的接收缓冲区的大小。默认和最小值为 0，允许操作系统或平台设置此值。对于大多数常见用例，建议使用默认值。

+   `TCP_SND_BUF_SIZE`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认 | 0 |
    | 范围 | 0 - 2G |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    确定在 TCP 传输器初始化期间设置的发送缓冲区的大小。默认和最小值为 0，允许操作系统或平台设置此值。��于大多数常见用例，建议使用默认值。

**重启类型。** 此部分中参数描述中使用的重启类型信息如下表所示：

**表 25.21 NDB 集群重启类型**

| 符号 | 重启类型 | 描述 |
| --- | --- | --- |
| N | 节点 | 可以使用滚动重启来更新该参数（参见第 25.6.5 节，“执行 NDB 集群的滚动重启”） |
| S | 系统 | 所有集群节点必须完全关闭，然后重新启动，以更改此参数 |
| I | 初始 | 必须使用`--initial`选项重新启动数据节点 |