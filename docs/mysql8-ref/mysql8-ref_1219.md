# 17.8.12 启用专用 MySQL 服务器的自动配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-dedicated-server.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-dedicated-server.html)

当启用`innodb_dedicated_server`时，`InnoDB`会自动配置以下变量：

+   `innodb_buffer_pool_size`

+   `innodb_redo_log_capacity`或在 MySQL 8.0.30 之前，`innodb_log_file_size`和`innodb_log_files_in_group`。

    注意

    `innodb_log_file_size`和`innodb_log_files_in_group`在 MySQL 8.0.30 中已弃用。这些变量已被`innodb_redo_log_capacity`变量取代。

+   `innodb_flush_method`

只有在 MySQL 实例驻留在可以使用所有可用系统资源的专用服务器上时才考虑启用`innodb_dedicated_server`。例如，如果在仅运行 MySQL 的 Docker 容器或专用 VM 中运行 MySQL 服务器，则考虑启用`innodb_dedicated_server`。如果 MySQL 实例与其他应用程序共享系统资源，则不建议启用`innodb_dedicated_server`。

接下���的信息描述了每个变量如何自动配置。

+   `innodb_buffer_pool_size`

    缓冲池大小根据服务器检测到的内存量进行配置。

    **表 17.8 自动配置的缓冲池大小**

    | 检测到的服务器内存 | 缓冲池大小 |
    | --- | --- |
    | 小于 1GB | 128MB（默认值） |
    | 1GB 至 4GB | *`检测到的服务器内存`* * 0.5 |
    | 大于 4GB | *`检测到的服务器内存`* * 0.75 |

+   `innodb_redo_log_capacity`

    重做日志容量根据服务器检测到的内存量进行配置，并且在某些情况下，根据是否显式配置`innodb_buffer_pool_size`。如果没有显式配置`innodb_buffer_pool_size`，则假定使用默认值。

    警告

    如果`innodb_buffer_pool_size`设置为大于检测到的服务器内存量的值，则自动重做日志容量配置行为是未定义的。

    **表 17.9 自动配置的日志文件大小**

    | 检测到的服务器内存 | 缓冲池大小 | 重做日志容量 |
    | --- | --- | --- |
    | 小于 1GB | 未配置 | 100MB |
    | 小于 1GB | 小于 1GB | 100MB |
    | 1GB 至 2GB | 不适用 | 100MB |
    | 2GB 至 4GB | 未配置 | 1GB |
    | 2GB 到 4GB | 任何配置的值 | round(0.5 * *`检测到的服务器内存`* in GB) * 0.5 GB |
    | 4GB 到 10.66GB | 不适用 | round(0.75 * *`检测到的服务器内存`* in GB) * 0.5 GB |
    | 10.66GB 到 170.66GB | 不适用 | round(0.5625 * *`检测到的服务器内存`* in GB) * 1 GB |
    | 大于 170.66GB | 不适用 | 128GB |

+   `innodb_log_file_size`（在 MySQL 8.0.30 中已弃用）

    日志文件大小根据自动配置的缓冲池大小进行配置。

    **表 17.10 自动配置的日志文件大小**

    | 缓冲池大小 | 日志文件大小 |
    | --- | --- |
    | 小于 8GB | 512MB |
    | 8GB 到 128GB | 1024MB |
    | 大于 128GB | 2048MB |

+   `innodb_log_files_in_group`（在 MySQL 8.0.30 中已弃用）

    日志文件数量根据自动配置的缓冲池大小进行配置。MySQL 8.0.14 中添加了自动配置`innodb_log_files_in_group`变量。

    **表 17.11 自动配置的日志文件数量**

    | 缓冲池大小 | ��志文件数量 |
    | --- | --- |
    | 小于 8GB | round(*`缓冲池大小`*) |
    | 8GB 到 128GB | round(*`缓冲池大小`* * 0.75) |
    | 大于 128GB | 64 |

    注意

    如果四舍五入的缓冲池大小值小于 2GB，则强制执行最小的`innodb_log_files_in_group`值为 2。

+   `innodb_flush_method`

    当启用`innodb_dedicated_server`时，刷新方法设置为`O_DIRECT_NO_FSYNC`。如果`O_DIRECT_NO_FSYNC`设置不可用，则使用默认的`innodb_flush_method`设置。

    `InnoDB`在刷新 I/O 期间使用`O_DIRECT`，但在每次写入操作后跳过`fsync()`系统调用。

    警告

    在 MySQL 8.0.14 之前，此设置不适用于需要`fsync()`系统调用来同步文件系统元数据更改的文件系统，如 XFS 和 EXT4。

    从 MySQL 8.0.14 开始，在创建新文件、增加文件大小和关闭文件后会调用`fsync()`，以确保文件系统元数据更改被同步。在每次写入操作后仍然跳过`fsync()`系统调用。

    如果重做日志文件和数据文件位于不同存储设备上，并且在数据文件写入从未备份电池支持的设备缓存之前发生意外退出，则可能会发生数据丢失。如果您使用或打算使用不同的存储设备用于重做日志文件和数据文件，并且您的数据文件位于没有备电池支持的设备缓存上，请改用`O_DIRECT`。

如果自动配置的选项在选项文件或其他地方明确配置，则使用明确指定的设置，并且类似于以下内容的启动警告将打印到`stderr`：

[警告] [000000] InnoDB: 由于明确指定了 innodb_buffer_pool_size=134217728，因此 innodb_dedicated_server 选项对 innodb_buffer_pool_size 无效。

对一个选项的显式配置不会阻止其他选项的自动配置。

如果启用了`innodb_dedicated_server`，并且明确配置了`innodb_buffer_pool_size`，则基于缓冲池大小配置的变量将使用根据服务器上检测到的内存量计算的缓冲池大小值，而不是明确定义的缓冲池大小值。

每次启动 MySQL 服务器时，自动配置的设置都会被评估和重新配置。
