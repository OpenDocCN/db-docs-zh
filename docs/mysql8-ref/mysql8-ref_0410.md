> 原文：[`dev.mysql.com/doc/refman/8.0/en/connection-control-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/connection-control-variables.html)

#### 8.4.2.2 连接控制系统和状态变量

本节描述了`CONNECTION_CONTROL`插件提供的系统和状态变量，以便配置和监视其操作。

+   连接控制系统变量

+   连接控制状态变量

##### 连接控制系统变量

如果安装了`CONNECTION_CONTROL`插件，则会公开这些系统变量：

+   `connection_control_failed_connections_threshold`

    | 命令行格式 | `--connection-control-failed-connections-threshold=#` |
    | --- | --- |
    | 系统变量 | `connection_control_failed_connections_threshold` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    在服务器在向帐户添加延迟以进行后续连接尝试之前允许的连续失败连接尝试次数：

    +   如果变量具有非零值*`N`*，则服务器从连续失败尝试*`N`*+1 开始添加延迟。如果帐户已达到连接响应被延迟的点，则下一个成功连接也会延迟。

    +   将此变量设置为零会禁用失败连接计数。在这种情况下，服务器永远不会添加延迟。

    有关`connection_control_failed_connections_threshold`与其他连接控制系统和状态变量的交互信息，请参见第 8.4.2.1 节，“连接控制插件安装”。

+   `connection_control_max_connection_delay`

    | 命令行格式 | `--connection-control-max-connection-delay=#` |
    | --- | --- |
    | 系统变量 | `connection_control_max_connection_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2147483647` |
    | 最小值 | `1000` |
    | 最大值 | `2147483647` |
    | 单位 | 毫秒 |

    如果`connection_control_failed_connections_threshold`大于零，则服务器响应失败连接尝试的最大延迟（以毫秒为单位）。

    有关`connection_control_max_connection_delay`如何与其他连接控制系统和状态变量交互的信息，请参见第 8.4.2.1 节，“连接控制插件安装”。

+   `connection_control_min_connection_delay`

    | 命令行格式 | `--connection-control-min-connection-delay=#` |
    | --- | --- |
    | 系统变量 | `connection_control_min_connection_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `1000` |
    | 最大值 | `2147483647` |
    | 单位 | 毫秒 |

    服务器响应失败连接尝试的最小延迟（以毫秒为单位），如果`connection_control_failed_connections_threshold`大于零。

    有关`connection_control_min_connection_delay`如何与其他连接控制系统和状态变量交互的信息，请参见第 8.4.2.1 节，“连接控制插件安装”。

##### 连接控制状态变量

如果安装了`CONNECTION_CONTROL`插件，则会公开此状态变量：

+   `Connection_control_delay_generated`

    服务器在响应失败连接尝试时添加延迟的次数。这不包括在达到由`connection_control_failed_connections_threshold`系统变量定义的阈值之前发生的尝试。

    这个变量提供了一个简单的计数器。要获取更详细的连接控制监控信息，请查看`INFORMATION_SCHEMA` `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 表；参见 Section 28.6.2, “The INFORMATION_SCHEMA CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS Table”。

    在运行时为`connection_control_failed_connections_threshold`赋值会将`Connection_control_delay_generated`重置为零。
