# 25.5.7 ndb_config — 提取 NDB 集群配置信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-config.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-config.html)

此工具从多个来源之一提取数据节点、SQL 节点和 API 节点的当前配置信息：NDB 集群管理节点，或其`config.ini`或`my.cnf`文件。默认情况下，管理节点是配置数据的来源；要覆盖默认设置，请使用`--config-file`或`--mycnf`选项执行 ndb_config。还可以通过指定其节点 ID 与`--config_from_node=*`node_id`*`来使用数据节点作为来源。

**ndb_config**还可以提供所有配置参数的离线转储，包括它们的默认值、最大值、最小值和其他信息。��储可以以文本或 XML 格式生成；有关更多信息，请参阅本节后面关于`--configinfo`和`--xml`选项的讨论。

你可以通过选项`--nodes`，`--system`，或`--connections`来按部分（`DB`，`SYSTEM`或`CONNECTIONS`）过滤结果。

所有可与**ndb_config**一起使用的选项都显示在下表中。表后面会有额外的描述。

**表 25.29 与程序 ndb_config 一起使用的命令行选项**

| 格式 | 描述 | 添加、弃用或移除 |
| --- | --- | --- |
| `--character-sets-dir=path` | 包含字符集的目录 | 移除：8.0.31 |
| `--cluster-config-suffix=name` | 在读取`my.cnf`文件中的 cluster_config 部分时覆盖默认组后缀；用于测试 | 添加：NDB 8.0.24 |
| `--config-binary-file=path/to/file` | 读取此二进制配置文件 | 添加：NDB 8.0.32 |
| `--config-file=file_name` | 设置到 config.ini 文件的路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--config-from-node=#` | 从具有此 ID 的节点（必须是数据节点）获取配置数据 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--configinfo` | 以文本格式转储所有 NDB 配置参数的信息，包括默认值、最大值和最小值。与--xml 一起使用以获取 XML 输出 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--connections` | 仅打印有关集群配置文件的[tcp]、[tcp default]、[sci]、[sci default]、[shm]或[shm default]部分指定的连接的信息。不能与--system 或--nodes 一起使用 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--connect-retries=#` | 在放弃之前重试连接的次数 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--connect-retry-delay=#` | 尝试联系管理服务器之间等待的秒数 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--connect-string=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--core-file` | 在错误时写入核心文件；用于调试 | 已移除：8.0.31 |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--defaults-group-suffix=string` | 还读取具有 concat(group, suffix)的组 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--diff-default` | 仅打印具有非默认值的配置参数 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--fields=string`,`-f` | 字段分隔符 | （在基于 MySQL 8.0 的所有 NDB 版本中支持） |
| `--help`,`-?` | 显示帮助文本并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--host=name` | 指定主机 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--login-path=path` | 从登录文件中读取给定路径 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--mycnf` | 从 my.cnf 文件中读取配置数据 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-connectstring=connection_string`,`-c connection_string` | 设置连接到 ndb_mgmd 的连接字符串。语法："[nodeid=id;][host=]hostname[:port]"。覆盖 NDB_CONNECTSTRING 和 my.cnf 中的条目 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-mgmd-host=connection_string`,`-c connection_string` | 与--ndb-connectstring 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--ndb-nodeid=#` | 为此节点设置节点 ID，覆盖--ndb-connectstring 设置的任何 ID | 移除：8.0.31 |
| `--ndb-optimized-node-selection` | 启用优化以选择事务节点。默认启用；使用--skip-ndb-optimized-node-selection 来禁用 | 移除：8.0.31 |
| `--no-defaults` | 不要从登录文件以外的任何选项文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--nodeid=#` | 获取具有此 ID 的节点配置 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--nodes` | 仅打印节点信息（集群配置文件的[ndbd]或[ndbd default]部分）。不能与--system 或--connections 一起使用 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--query=string`,`-q string` | 一个或多个查询选项（属性） | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--query-all`,`-a` | 将所有参数和值转储到单个逗号分隔的字符串中 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--print-defaults` | 打印程序参数列表并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--rows=string`,`-r string` | 行分隔符 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--system` | 仅打印 SYSTEM 部分信息（请参阅 ndb_config --configinfo 输出）。不能与 --nodes 或 --connections 一起使用 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--type=name` | 指定节点类型 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--usage`,`-?` | 显示帮助文本并退出；与 --help 相同 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--version`,`-V` | 显示版本信息并退出 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--configinfo --xml` | 使用 --xml ��� --configinfo 以 XML 格式获取所有 NDB 配置参数的转储，包括默认、最大和最小值 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| 格式 | 描述 | 添加、弃用或移除 |

+   `cluster-config-suffix`

    | 命令行格式 | `--cluster-config-suffix=name` |
    | --- | --- |
    | 引入 | 8.0.24-ndb-8.0.24 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在读取`my.cnf`中的集群配置部分时覆盖默认组后缀；用于测试。

+   `--configinfo`

    `--configinfo`选项导致**ndb_config**列出了 NDB 集群配置参数的列表，这些参数受 NDB 集群分发的支持，其中**ndb_config**是其中的一部分，包括以下信息：

    +   每个参数的目的、效果和用法的简要描述

    +   参数可能在`config.ini`文件中使用的部分

    +   参数的数据类型或计量单位

    +   在适用的情况下，参数的默认值、最小值和最大值

    +   NDB Cluster 发布版本和构建信息

    默认情况下，此输出为文本格式。这里显示了部分输出：

    ```sql
    $> ndb_config --configinfo

    ****** SYSTEM ******

    Name (String)
    Name of system (NDB Cluster)
    MANDATORY

    PrimaryMGMNode (Non-negative Integer)
    Node id of Primary ndb_mgmd(MGM) node
    Default: 0 (Min: 0, Max: 4294967039)

    ConfigGenerationNumber (Non-negative Integer)
    Configuration generation number
    Default: 0 (Min: 0, Max: 4294967039)

    ****** DB ******

    MaxNoOfSubscriptions (Non-negative Integer)
    Max no of subscriptions (default 0 == MaxNoOfTables)
    Default: 0 (Min: 0, Max: 4294967039)

    MaxNoOfSubscribers (Non-negative Integer)
    Max no of subscribers (default 0 == 2 * MaxNoOfTables)
    Default: 0 (Min: 0, Max: 4294967039)

    …
    ```

    与`--xml`选项一起使用以获取 XML 格式的输出。

+   `--config-binary-file=*`path-to-file`*`

    | 命令行格式 | `--config-binary-file=path/to/file` |
    | --- | --- |
    | 引入版本 | 8.0.32-ndb-8.0.32 |
    | 类型 | 文件名 |
    | 默认值 |  |

    给出管理服务器缓存的二进制配置文件（`ndb_*`nodeID`*_config.bin.*`seqno`*`）的路径。这可以是相对路径或绝对路径。如果管理服务器和使用的**ndb_config**二进制文件位于不同主机上，则必须使用绝对路径。

    这个示例演示了将`--config-binary-file`与其他**ndb_config**选项结合使用以获得有用的输出：

    ```sql
    > ndb_config --config-binary-file=ndb_50_config.bin.1 --diff-default --type=ndbd
    config of [DB] node id 5 that is different from default 
    CONFIG_PARAMETER,ACTUAL_VALUE,DEFAULT_VALUE 
    NodeId,5,(mandatory) 
    BackupDataDir,/home/jon/data/8.0,(null) 
    DataDir,/home/jon/data/8.0,. 
    DataMemory,2G,98M 
    FileSystemPath,/home/jon/data/8.0,(null) 
    HostName,127.0.0.1,localhost 
    Nodegroup,0,(null) 
    ThreadConfig,,(null) 

    config of [DB] node id 6 that is different from default 
    CONFIG_PARAMETER,ACTUAL_VALUE,DEFAULT_VALUE 
    NodeId,6,(mandatory) 
    BackupDataDir,/home/jon/data/8.0,(null) 
    DataDir,/home/jon/data/8.0,. 
    DataMemory,2G,98M 
    FileSystemPath,/home/jon/data/8.0,(null) 
    HostName,127.0.0.1,localhost 
    Nodegroup,0,(null) 
    ThreadConfig,,(null)

    > ndb_config --config-binary-file=ndb_50_config.bin.1 --diff-default --system
    config of [SYSTEM] system 
    CONFIG_PARAMETER,ACTUAL_VALUE,DEFAULT_VALUE 
    Name,MC_20220216092809,(mandatory) 
    ConfigGenerationNumber,1,0 
    PrimaryMGMNode,50,0
    ```

    显示了`config.ini`文件的相关部分：

    ```sql
    [ndbd default]
    DataMemory= 2G
    NoOfReplicas= 2

    [ndb_mgmd]
    NodeId= 50
    HostName= 127.0.0.1

    [ndbd]
    NodeId= 5
    HostName= 127.0.0.1
    DataDir= /home/jon/data/8.0

    [ndbd]
    NodeId= 6
    HostName= 127.0.0.1
    DataDir= /home/jon/data/8.0
    ```

    通过将输出与配置文件进行比较，您可以看到管理服务器已将文件中的所有设置写入二进制缓存，从而应用于集群。

+   `--config-file=*`path-to-file`*`

    | 命令行格式 | `--config-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 |  |

    给出集群配置文件（`config.ini`）的路径。这可以是相对路径或绝对路径。如果管理服务器和使用的**ndb_config**二进制文件位于不同主机上，则必须使用绝对路径。

+   `--config_from_node=#`

    | 命令行格式 | `--config-from-node=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `none` |
    | 最小值 | `1` |
    | 最大值 | `48` |

    从具有此 ID 的数据节点获取集群的配置数据。

    如果具有此 ID 的节点不是数据节点，则**ndb_config**将失败并显示错误。（要从管理节点而不是数据节点获取配置数据，只需省略此选项。）

+   `--connections`

    | 命令行格式 | `--connections` |
    | --- | --- |

    告诉**ndb_config**仅打印`CONNECTIONS`信息，即集群配置文件的`[tcp]`、`[tcp default]`、`[shm]`或`[shm default]`部分中找到的参数信息（有关更多信息，请参见第 25.4.3.10 节，“NDB Cluster TCP/IP Connections”和第 25.4.3.12 节，“NDB Cluster Shared-Memory Connections”）。

    此选项与`--nodes`和`--system`互斥；这 3 个选项中只能使用一个。

+   `--diff-default`

    | 命令行格式 | `--diff-default` |
    | --- | --- |

    仅打印具有非默认值的配置参数。

+   `--fields=*`delimiter`*`, `-f` *`delimiter`*

    | 命令行格式 | `--fields=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    指定用于分隔结果中字段的*`delimiter`*字符串。默认值为`,`（逗号字符）。

    注意

    如果*`delimiter`*包含空格或转义字符（例如换行符`\n`），则必须用引号括起来。

+   `--host=*`hostname`*`

    | 命令行格式 | `--host=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    指定要获取配置信息的节点的主机名。

    注意

    虽然主机名`localhost`通常解析为 IP 地址`127.0.0.1`，但这并不一定适用于所有操作平台和配置。这意味着当`localhost`在`config.ini`中使用时，如果在解析为不同地址的主机上运行**ndb_config `--host=localhost`**，可能会导致失败（例如，在某些版本的 SUSE Linux 上，这是`127.0.0.2`）。一般来说，为了获得最佳结果，您应该对所有与主机相关的 NDB Cluster 配置值使用数字 IP 地址，或验证所有 NDB Cluster 主机以相同方式处理`localhost`。

+   `--mycnf`

    | 命令行格式 | `--mycnf` |
    | --- | --- |

    从`my.cnf`文件中读取配置数据。

+   `--ndb-connectstring=*`connection_string`*`, `-c *`connection_string`*`

    | 命令行格式 | `--ndb-connectstring=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    指定用于连接到管理服务器的连接字符串。连接字符串的格式与第 25.4.3.3 节，“NDB 集群连接字符串”中描述的相同，并默认为`localhost:1186`。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要从登录文件以外的任何选项文件中读取默认选项。

+   `--nodeid=*`node_id`*`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 已移除 | 8.0.31 |
    | 类型 | 整数 |
    | 默认值 | `[none]` |

    指定要获取配置信息的节点的节点 ID。

+   `--nodes`

    | 命令行格式 | `--nodes` |
    | --- | --- |

    告诉**ndb_config**仅打印与集群配置文件的`[ndbd]`或`[ndbd default]`部分中定义的参数相关的信息（请参见第 25.4.3.6 节，“定义 NDB 集群数据节点”）。

    此选项与`--connections`和`--system`互斥；只能使用这 3 个选项中的一个。

+   `--query=*`query-options`*`，`-q` *`query-options`*

    | 命令行格式 | `--query=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    这是一个查询选项的逗号分隔列表，即要返回的一个或多个节点属性的列表。这些包括`nodeid`（节点 ID）、type（节点类型，即`ndbd`、`mysqld`或`ndb_mgmd`）以及要获取其值的任何配置参数。

    例如，`--query=nodeid,type,datamemory,datadir` 返回每个节点的节点 ID、节点类型、`DataMemory` 和 `DataDir`。

    注意

    如果给定参数不适用于某种类型的节点，则对应值将返回为空字符串。有关更多信息，请参见本节后面的示例。

+   `--query-all`，`-a`

    | 命令行格式 | `--query-all` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    返回所有查询选项（节点属性）的逗号分隔列表（请注意，此列表是一个字符串）。

+   `--rows=*`separator`*`，`-r` *`separator`*

    | 命令行格式 | `--rows=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 |  |

    指定用于在结果中分隔行的*`separator`*字符串。默认为一个空格字符。

    注意

    如果*`separator`*包含空格或转义字符（如`\n`代表换行符），则必须用引号括起来。

+   `--system`

    | 命令行格式 | `--system` |
    | --- | --- |

    告诉**ndb_config**仅打印`SYSTEM`信息。这包括无法在运行时更改的系统变量；因此，在集群配置文件的相应部分中没有对应的内容。它们可以在**ndb_config** `--configinfo`的输出中看到（前缀为`****** SYSTEM ******`）。

    此选项与`--nodes`和`--connections`互斥；只能使用这 3 个选项中的一个。

+   `--type=*`node_type`*`

    | 命令行格式 | `--type=name` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `[none]` |
    | 有效值 | `ndbd``mysqld``ndb_mgmd` |

    仅返回适用于指定*`node_type`*（`ndbd`、`mysqld`或`ndb_mgmd`）的节点的配置值。

+   `--usage`、`--help`或`-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    使**ndb_config**打印可用选项列表，然后退出。

+   `--version`、`-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    使**ndb_config**打印版本信息字符串，然后退出。

+   `--configinfo` `--xml`

    | 命令行格式 | `--configinfo --xml` |
    | --- | --- |

    通过添加此选项，使**ndb_config** `--configinfo`以 XML 格式提供输出。此示例中显示了部分输出：

    ```sql
    $> ndb_config --configinfo --xml

    <configvariables protocolversion="1" ndbversionstring="5.7.44-ndb-7.5.32"
                        ndbversion="460032" ndbversionmajor="7" ndbversionminor="5"
                        ndbversionbuild="0">
      <section name="SYSTEM">
        <param name="Name" comment="Name of system (NDB Cluster)" type="string"
                  mandatory="true"/>
        <param name="PrimaryMGMNode" comment="Node id of Primary ndb_mgmd(MGM) node"
                  type="unsigned" default="0" min="0" max="4294967039"/>
        <param name="ConfigGenerationNumber" comment="Configuration generation number"
                  type="unsigned" default="0" min="0" max="4294967039"/>
      </section>
      <section name="MYSQLD" primarykeys="NodeId">
        <param name="wan" comment="Use WAN TCP setting as default" type="bool"
                  default="false"/>
        <param name="HostName" comment="Name of computer for this node"
                  type="string" default=""/>
        <param name="Id" comment="NodeId" type="unsigned" mandatory="true"
                  min="1" max="255" deprecated="true"/>
        <param name="NodeId" comment="Number identifying application node (mysqld(API))"
                  type="unsigned" mandatory="true" min="1" max="255"/>
        <param name="ExecuteOnComputer" comment="HostName" type="string"
                  deprecated="true"/>

        …

      </section>

      …

    </configvariables>
    ```

    注意

    通常，由**ndb_config** `--configinfo` `--xml`生成的 XML 输出使用每个元素一行的格式；我们在前面的示例以及下一个示例中添加了额外的空格，以提高可读性。这不应对使用此输出的应用程序产生任何影响，因为大多数 XML 处理器要么默认忽略非必要的空格，要么可以被指示忽略。

    XML 输出还指示了当更改给定参数需要使用`--initial`选项重新启动数据节点时。这通过相应的`<param>`元素中存在`initial="true"`属性来显示。此外，还显示了重新启动类型（`system`或`node`）；如果某个参数需要系统重新启动，则在相应的`<param>`元素中存在`restart="system"`属性。例如，更改`Diskless`参数设置的值需要进行系统初始重新启动，如下所示（高亮显示`restart`和`initial`属性以便查看）：

    ```sql
    <param name="Diskless" comment="Run wo/ disk" type="bool" default="false"
              *restart="system" initial="true"*/>
    ```

    目前，在不需要初始重新启动的参数对应的`<param>`元素的 XML 输出中不包含`initial`属性；换句话说，`initial="false"`是默认值，如果属性不存在，则应假定值为`false`。类似地，默认的重新启动类型是`node`（即集群的在线或“滚动”重新启动），但只有当重新启动类型为`system`时才包含`restart`属性（表示所有集群节点必须同时关闭，然后重新启动）。

    弃用的参数在 XML 输出中通过`deprecated`属性来指示，如下所示：

    ```sql
    <param name="NoOfDiskPagesToDiskAfterRestartACC" comment="DiskCheckpointSpeed"
           type="unsigned" default="20" min="1" max="4294967039" *deprecated="true"*/>
    ```

    在这种情况下，`comment`指的是一个或多个取代弃用参数的参数。与`initial`类似，`deprecated`属性仅在参数被弃用时显示为`deprecated="true"`，对于未被弃用的参数根本不显示。 （Bug #21127135）

    必需的参数通过`mandatory="true"`来指示，如下所示：

    ```sql
    <param name="NodeId"
              comment="Number identifying application node (mysqld(API))"
              type="unsigned" *mandatory="true"* min="1" max="255"/>
    ```

    与`initial`或`deprecated`属性仅对需要初始重新启动或已弃用的参数显示类似，`mandatory`属性仅在给定参数实际上是必需的时包含。

    重要

    `--xml`选项只能与`--configinfo`选项一起使用。在没有`--configinfo`的情况下使用`--xml`会导致错误。

    与此程序一起使用的选项用于获取当前配置数据，`--configinfo` 和 `--xml` 使用编译 **ndb_config** 时从 NDB Cluster 源获取的信息。因此，这两个选项不需要连接到运行中的 NDB Cluster 或访问 `config.ini` 或 `my.cnf` 文件。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印程序参数列表并退出。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    仅从给定文件中读取默认选项。

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    在读取全局文件后读取给定文件。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    同时读取带有 concat(group, suffix) 的组。

+   `--login-path`

    | 命令行格式 | `--login-path=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    从登录文件中读取给定路径。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助文本并退出。

+   `--connect-string`

    | 命令行格式 | `--connect-string=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    与 `--ndb-connectstring` 相同。

+   `--ndb-mgmd-host`

    | 命令行格式 | `--ndb-mgmd-host=connection_string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    与 `--ndb-connectstring` 相同。

+   `--ndb-nodeid`

    | 命令行格式 | `--ndb-nodeid=#` |
    | --- | --- |
    | 已移除 | 8.0.31 |
    | 类型 | 整数 |
    | 默认值 | `[无]` |

    为此节点设置节点 ID，覆盖由 `--ndb-connectstring` 设置的任何 ID。

+   `--core-file`

    | 命令行格式 | `--core-file` |
    | --- | --- |
    | 已移除 | 8.0.31 |

    在错误时写入核心文件；用于调试。

+   `--character-sets-dir`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 移除 | 8.0.31 |

    包含字符集的目录。

+   `--connect-retries`

    | 命令行格式 | `--connect-retries=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `12` |
    | 最小值 | `0` |
    | 最大值 | `12` |

    在放弃之前重试连接的次数。

+   `--connect-retry-delay`

    | 命令行格式 | `--connect-retry-delay=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `5` |
    | 最小值 | `0` |
    | 最大值 | `5` |

    每次尝试联系管理服务器之间等待的秒数。

+   `--ndb-optimized-node-selection`

    | 命令行格式 | `--ndb-optimized-node-selection` |
    | --- | --- |
    | 移除 | 8.0.31 |

    启用优化以选择事务节点。默认启用；使用`--skip-ndb-optimized-node-selection`来禁用。

结合其他**ndb_config**选项（如`--query`或`--type`)与`--configinfo`（带有或不带有`--xml`选项）不受支持。当前，如果尝试这样做，通常的结果是除了`--configinfo`或`--xml`之外的所有其他选项都会被简单忽略。*但是，此行为不能保证且随时可能更改*。此外，由于**ndb_config**在使用`--configinfo`选项时不访问 NDB 集群或读取任何文件，因此尝试使用`--configinfo`指定其他选项，如`--ndb-connectstring`或`--config-file`没有任何意义。

#### 示例

1.  获得集群中每个节点的节点 ID 和类型：

    ```sql
    $> ./ndb_config --query=nodeid,type --fields=':' --rows='\n'
    1:ndbd
    2:ndbd
    3:ndbd
    4:ndbd
    5:ndb_mgmd
    6:mysqld
    7:mysqld
    8:mysqld
    9:mysqld
    ```

    在这个例子中，我们使用`--fields`选项用冒号字符(`:`)分隔每个节点的 ID 和类型，并使用`--rows`选项将每个节点的值放在输出中的新行上。

1.  生成一个连接字符串，可供数据、SQL 和 API 节点连接到管理服务器：

    ```sql
    $> ./ndb_config --config-file=usr/local/mysql/cluster-data/config.ini \
    --query=hostname,portnumber --fields=: --rows=, --type=ndb_mgmd
    198.51.100.179:1186
    ```

1.  这个调用**ndb_config**仅检查数据节点（使用`--type`选项），并显示每个节点的 ID 和主机名的值，以及为其设置的`DataMemory`和`DataDir`参数的值：

    ```sql
    $> ./ndb_config --type=ndbd --query=nodeid,host,datamemory,datadir -f ' : ' -r '\n'
    1 : 198.51.100.193 : 83886080 : /usr/local/mysql/cluster-data
    2 : 198.51.100.112 : 83886080 : /usr/local/mysql/cluster-data
    3 : 198.51.100.176 : 83886080 : /usr/local/mysql/cluster-data
    4 : 198.51.100.119 : 83886080 : /usr/local/mysql/cluster-data
    ```

    在这个例子中，我们使用了短选项`-f`和`-r`来设置字段分隔符和行分隔符，同时使用了短选项`-q`来传递要获取的参数列表。

1.  要排除除特定主机之外的任何主机的结果，请使用`--host`选项：

    ```sql
    $> ./ndb_config --host=198.51.100.176 -f : -r '\n' -q id,type
    3:ndbd
    5:ndb_mgmd
    ```

    在这个例子中，我们还使用了短形式`-q`来确定要查询的属性。

    同样，您可以使用`--nodeid`选项将结果限制为具有特定 ID 的节点。
