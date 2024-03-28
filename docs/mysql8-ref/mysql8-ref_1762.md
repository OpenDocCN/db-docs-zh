> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-config-values.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-config-values.html)

#### 25.6.16.11 ndbinfo config_values 表

`config_values`表提供有关节点配置参数值的当前状态的信息。表中的每一行对应于给定节点上参数的当前值。

`config_values`表包含以下列：

+   `node_id`

    集群中节点的 ID

+   `config_param`

    参数的内部 ID 编号

+   `config_value`

    参数的当前值

##### 注意

此表的`config_param`列和`config_params`表的`param_number`列使用相同的参数标识符。通过在这些列上连接两个表，您可以获取有关所需节点配置参数的详细信息。此处显示的查询提供了集群中每个数据节点的所有参数的当前值，按节点 ID 和参数名称排序：

```sql
SELECT    v.node_id AS 'Node Id',
          p.param_name AS 'Parameter',
          v.config_value AS 'Value'
FROM      config_values v
JOIN      config_params p
ON        v.config_param=p.param_number
WHERE     p.param_name NOT LIKE '\_\_%'
ORDER BY  v.node_id, p.param_name;
```

在用于简单测试的小型示例集群上运行上一个查询的部分输出：

```sql
+---------+------------------------------------------+----------------+
| Node Id | Parameter                                | Value          |
+---------+------------------------------------------+----------------+
|       2 | Arbitration                              | 1              |
|       2 | ArbitrationTimeout                       | 7500           |
|       2 | BackupDataBufferSize                     | 16777216       |
|       2 | BackupDataDir                            | /home/jon/data |
|       2 | BackupDiskWriteSpeedPct                  | 50             |
|       2 | BackupLogBufferSize                      | 16777216       |

...

|       3 | TotalSendBufferMemory                    | 0              |
|       3 | TransactionBufferMemory                  | 1048576        |
|       3 | TransactionDeadlockDetectionTimeout      | 1200           |
|       3 | TransactionInactiveTimeout               | 4294967039     |
|       3 | TwoPassInitialNodeRestartCopy            | 0              |
|       3 | UndoDataBuffer                           | 16777216       |
|       3 | UndoIndexBuffer                          | 2097152        |
+---------+------------------------------------------+----------------+
248 rows in set (0.02 sec)
```

`WHERE`子句过滤掉以双下划线(`__`)开头的参数；这些参数是为 NDB 开发人员的测试和其他内部用途保留的，并不打算在生产 NDB 集群中使用。

通过发出适当的查询，您可以获得更具体、更详细或两者兼具的输出。此示例提供了有关集群中所有数据节点当前设置的`NodeId`、`NoOfReplicas`、`HostName`、`DataMemory`、`IndexMemory`和`TotalSendBufferMemory`参数的所有类型的可用信息：

```sql
SELECT  p.param_name AS Name,
        v.node_id AS Node,
        p.param_type AS Type,
        p.param_default AS 'Default',
        p.param_min AS Minimum,
        p.param_max AS Maximum,
        CASE p.param_mandatory WHEN 1 THEN 'Y' ELSE 'N' END AS 'Required',
        v.config_value AS Current
FROM    config_params p
JOIN    config_values v
ON      p.param_number = v.config_param
WHERE   p. param_name
  IN ('NodeId', 'NoOfReplicas', 'HostName',
      'DataMemory', 'IndexMemory', 'TotalSendBufferMemory')\G
```

在用于简单测试的具有 2 个数据节点的小型 NDB 集群上运行此查询的输出如下所示：

```sql
*************************** 1\. row ***************************
    Name: NodeId
    Node: 2
    Type: unsigned
 Default:
 Minimum: 1
 Maximum: 144
Required: Y
 Current: 2
*************************** 2\. row ***************************
    Name: HostName
    Node: 2
    Type: string
 Default: localhost
 Minimum:
 Maximum:
Required: N
 Current: 127.0.0.1
*************************** 3\. row ***************************
    Name: TotalSendBufferMemory
    Node: 2
    Type: unsigned
 Default: 0
 Minimum: 262144
 Maximum: 4294967039
Required: N
 Current: 0
*************************** 4\. row ***************************
    Name: NoOfReplicas
    Node: 2
    Type: unsigned
 Default: 2
 Minimum: 1
 Maximum: 4
Required: N
 Current: 2
*************************** 5\. row ***************************
    Name: DataMemory
    Node: 2
    Type: unsigned
 Default: 102760448
 Minimum: 1048576
 Maximum: 1099511627776
Required: N
 Current: 524288000
*************************** 6\. row ***************************
    Name: NodeId
    Node: 3
    Type: unsigned
 Default:
 Minimum: 1
 Maximum: 144
Required: Y
 Current: 3
*************************** 7\. row ***************************
    Name: HostName
    Node: 3
    Type: string
 Default: localhost
 Minimum:
 Maximum:
Required: N
 Current: 127.0.0.1
*************************** 8\. row ***************************
    Name: TotalSendBufferMemory
    Node: 3
    Type: unsigned
 Default: 0
 Minimum: 262144
 Maximum: 4294967039
Required: N
 Current: 0
*************************** 9\. row ***************************
    Name: NoOfReplicas
    Node: 3
    Type: unsigned
 Default: 2
 Minimum: 1
 Maximum: 4
Required: N
 Current: 2
*************************** 10\. row ***************************
    Name: DataMemory
    Node: 3
    Type: unsigned
 Default: 102760448
 Minimum: 1048576
 Maximum: 1099511627776
Required: N
 Current: 524288000 10 rows in set (0.01 sec)
```
