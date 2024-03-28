> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-system-definition.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-system-definition.html)

#### 25.4.3.8 定义系统

`[system]` 部分用于适用于整个集群的参数。 `Name` 系统参数用于 MySQL Enterprise Monitor；`ConfigGenerationNumber` 和 `PrimaryMGMNode` 在生产环境中不使用。 除非使用 NDB 集群与 MySQL Enterprise Monitor，否则不需要在 `config.ini` 文件中有 `[system]` 部分。

关于这些参数的更多信息可以在以下列表中找到：

+   `ConfigGenerationNumber`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | 0 |
    | 范围 | 0 - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    配置生成编号。当前未使用此参数。

+   `Name`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字符串 |
    | 默认值 | [...] |
    | 范围 | ... |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    为集群设置名称。此参数对于使用 MySQL Enterprise Monitor 进行部署是必需的；否则不使用。

    您可以通过检查 `Ndb_system_name` 状态变量来获取此参数的值。在 NDB API 应用程序中，您还可以使用 `get_system_name()` 进行检索。

+   `PrimaryMGMNode`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | 0 |
    | 范围 | 0 - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    主管理节点的节点 ID。当前未使用此参数。

**重启类型。** 此部分中参数描述中使用的重启类型的信息显示在以下表中：

**表 25.19 NDB 集群重启类型**

| 符号 | 重启类型 | 描述 |
| --- | --- | --- |
| N | 节点 | 可以使用滚动重启来更新该参数（参见第 25.6.5 节，“执行 NDB 集群的滚动重启”） |
| S | 系统 | 所有集群节点必须完全关闭，然后重新启动，以更改此参数 |
| I | 初始 | 必须使用`--initial`选项重新启动数据节点 |
