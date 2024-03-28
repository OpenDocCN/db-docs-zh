# 6.2.8 连接压缩控制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/connection-compression-control.html`](https://dev.mysql.com/doc/refman/8.0/en/connection-compression-control.html)

与服务器的连接可以在客户端和服务器之间的流量上使用压缩，以减少连接发送的字节数。默认情况下，连接是未压缩的，但如果服务器和客户端就一个相互允许的压缩算法达成一致，则可以进行压缩。

压缩连接起源于客户端，但会影响客户端和服务器双方的 CPU 负载，因为双方都会执行压缩和解压操作。由于启用压缩会降低性能，其好处主要体现在网络带宽较低、网络传输时间主导压缩和解压操作成本、结果集较大的情况下。

本节描述了可用的压缩控制配置参数以及用于监视压缩使用的信息来源。适用于经典的 MySQL 协议连接。

压缩控制适用于客户端程序连接到服务器以及参与源/副本复制或组复制的服务器之间的连接。压缩控制不适用于 `FEDERATED` 表的连接。在以下讨论中，“客户端连接”是指从任何支持压缩的源发起的连接到服务器的连接，除非上下文指示特定的连接类型。

注意

MySQL 8.0.19 开始，X 协议连接到 MySQL 服务器实例支持压缩，但 X 协议连接的压缩与此处描述的经典 MySQL 协议连接的压缩独立操作，并且受到单独控制。有关 X 协议连接压缩的信息，请参见 第 22.5.5 节，“使用 X 插件进行连接压缩”。

+   配置连接压缩

+   配置传统连接压缩

+   监视连接压缩

#### 配置连接压缩

从 MySQL 8.0.18 开始，可以使用以下配置参数来控制连接压缩：

+   `protocol_compression_algorithms` 系统变量配置了服务器允许用于传入连接的压缩算法。

+   `--compression-algorithms` 和 `--zstd-compression-level` 命令行选项配置了以下客户端程序所允许的压缩算法和`zstd`压缩级别：**mysql**, **mysqladmin**, **mysqlbinlog**, **mysqlcheck**, **mysqldump**, **mysqlimport**, **mysqlpump**, **mysqlshow**, **mysqlslap**, 和 **mysqltest**，以及 **mysql_upgrade**。从 MySQL 8.0.20 版本开始，MySQL Shell 也提供这些命令行选项。

+   `mysql_options()` 函数的 `MYSQL_OPT_COMPRESSION_ALGORITHMS` 和 `MYSQL_OPT_ZSTD_COMPRESSION_LEVEL` 选项配置了使用 MySQL C API 的客户端程序所允许的压缩算法和`zstd`压缩级别。

+   `CHANGE MASTER TO` 语句的 `MASTER_COMPRESSION_ALGORITHMS` 和 `MASTER_ZSTD_COMPRESSION_LEVEL` 选项配置了参与源/复制复制的副本服务器所允许的压缩算法和`zstd`压缩级别。从 MySQL 8.0.23 版本开始，使用 `CHANGE REPLICATION SOURCE TO` 语句和选项 `SOURCE_COMPRESSION_ALGORITHMS` 和 `SOURCE_ZSTD_COMPRESSION_LEVEL`。

+   `group_replication_recovery_compression_algorithms` 和 `group_replication_recovery_zstd_compression_level` 系统变量配置了在新成员加入组并连接到提供者时，Group Replication 恢复连接所允许的压缩算法和`zstd`压缩级别。

允许指定压缩算法的配置参数是字符串值，并接受一个或多个逗号分隔的压缩算法名称列表，顺序不限��可从以下项目中选择（不区分大小写）：

+   `zlib`: 允许使用`zlib`压缩算法的连接。

+   `zstd`: 允许使用`zstd`压缩算法的连接。

+   `uncompressed`：允许未压缩的连接。

注意

因为`uncompressed`是一个可能已配置或未配置的算法名称，因此可以配置 MySQL *不*允许未压缩的连接。

示例：

+   要配置服务器允许传入连接使用的压缩算法，请设置`protocol_compression_algorithms`系统变量。默认情况下，服务器允许所有可用算法。要在启动时明确配置该设置，请在服务器的`my.cnf`文件中使用以下行：

    ```sql
    [mysqld]
    protocol_compression_algorithms=zlib,zstd,uncompressed
    ```

    要在运行时设置和持久化`protocol_compression_algorithms`系统变量为该值，请使用以下语句：

    ```sql
    SET PERSIST protocol_compression_algorithms='zlib,zstd,uncompressed';
    ```

    `SET PERSIST`为运行中的 MySQL 实例设置一个值。它还保存该值，导致其在随后的服务器重新启动时保留。要更改运行中的 MySQL 实例的值，而不使其在随后的重新启动中保留，使用`GLOBAL`关键字而不是`PERSIST`。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。

+   要仅允许使用`zstd`压缩的传入连接，请在启动时像这样配置服务器：

    ```sql
    [mysqld]
    protocol_compression_algorithms=zstd
    ```

    或者，在运行时进行更改：

    ```sql
    SET PERSIST protocol_compression_algorithms='zstd';
    ```

+   要允许**mysql**客户端发起`zlib`或`uncompressed`连接，请这样调用它：

    ```sql
    mysql --compression-algorithms=zlib,uncompressed
    ```

+   要配置副本使用`zlib`或`zstd`连接连接到源，对于`zstd`连接使用压缩级别 7，请使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）：

    ```sql
    CHANGE REPLICATION SOURCE TO
      SOURCE_COMPRESSION_ALGORITHMS = 'zlib,zstd',
      SOURCE_ZSTD_COMPRESSION_LEVEL = 7;
    ```

    这假定`replica_compressed_protocol`或`slave_compressed_protocol`系统变量已禁用，原因在配置传统连接压缩中有描述。

为了成功建立连接，连接的双方必须就一种互相允许的压缩算法达成一致。算法协商过程尝试使用`zlib`，然后是`zstd`，最后是`uncompressed`。如果双方找不到共同的算法，连接尝试将失败。

因为双方必须就压缩算法达成一致，而`uncompressed`是一个不一定被允许的算法值，所以回退到未压缩连接并不一定发生。例如，如果服务器配置为允许`zstd`，而客户端配置为允许`zlib,uncompressed`，则客户端根本无法连接。在这种情况下，双方没有共同的算法，因此连接尝试失败。

启用指定`zstd`压缩级别的配置参数采用从 1 到 22 的整数值，较大的值表示更高级别的压缩。默认的`zstd`压缩级别为 3。压缩级别设置对不使用`zstd`压缩的连接没有影响。

可配置的`zstd`压缩级别可在减少网络流量和增加 CPU 负载与增加网络流量和降低 CPU 负载之间进行选择。更高的压缩级别可以减少网络拥塞，但额外的 CPU 负载可能会降低服务器性能。

#### 配置遗留连接压缩

在 MySQL 8.0.18 之前，这些配置参数可用于控制连接压缩：

+   客户端程序支持`--compress`命令行选项，以指定与服务器的连接使用压缩。

+   对于使用 MySQL C API 的程序，通过为`mysql_options()`函数启用`MYSQL_OPT_COMPRESS`选项，指定与服务器的连接使用压缩。

+   对于源/副本复制，启用系统变量`replica_compressed_protocol`（从 MySQL 8.0.26 开始）或`slave_compressed_protocol`（在 MySQL 8.0.26 之前）指定副本连接到源的压缩使用。

在每种情况下，当指定使用压缩时，如果双方都允许，连接将使用`zlib`压缩算法，否则将回退到未压缩连接。

截至 MySQL 8.0.18，刚刚描述的压缩参数成为遗留参数，因为引入了用于更好控制连接压缩的其他压缩参数，详细信息请参阅配置连接压缩。一个例外是 MySQL Shell，在那里，`--compress`命令行选项仍然有效，并且可以用于请求压缩而不选择压缩算法。有关 MySQL Shell 的连接压缩控制信息，请参阅使用压缩连接。

旧压缩参数与新参数互动，并且它们的语义如下更改：

+   遗留的`--compress`选项的含义取决于是否指定了`--compression-algorithms`：

    +   当未指定`--compression-algorithms`时，`--compress`等同于指定一个客户端算法集为`zlib,uncompressed`。

    +   当指定`--compression-algorithms`时，`--compress`等同于指定一个`zlib`算法集，完整的客户端算法集是`zlib`加上`--compression-algorithms`指定的算法的并集。例如，使用`--compress`和`--compression-algorithms=zlib,zstd`，允许的算法集是`zlib`加上`zlib,zstd`；即，`zlib,zstd`。使用`--compress`和`--compression-algorithms=zstd,uncompressed`，允许的算法集是`zlib`加上`zstd,uncompressed`；即，`zlib,zstd,uncompressed`。

+   对于`mysql_options()` C API 函数，遗留的`MYSQL_OPT_COMPRESS`选项和`MYSQL_OPT_COMPRESSION_ALGORITHMS`选项之间发生相同类型的交互。

+   如果启用了`replica_compressed_protocol`或`slave_compressed_protocol`系统变量，则优先于`MASTER_COMPRESSION_ALGORITHMS`，如果源和复制品都允许该算法，则连接到源使用`zlib`压缩。如果禁用了`replica_compressed_protocol`或`slave_compressed_protocol`，则`MASTER_COMPRESSION_ALGORITHMS`的值适用。

注意

遗留的压缩控制参数自 MySQL 8.0.18 起已弃用；预计将在未来的 MySQL 版本中移除。

#### 监控连接压缩

`Compression`状态变量为`ON`或`OFF`，表示当前连接是否使用了压缩。

**mysql**客户端的`\status`命令显示一行，如果当前连接启用了压缩，则会显示`Protocol: Compressed`。如果该行不存在，则连接未压缩。

从 8.0.14 开始，MySQL Shell 的`\status`命令显示一个`Compression:`行，指示连接是否压缩，显示为`Disabled`或`Enabled`。

截至 MySQL 8.0.18，有以下额外的信息源可用于监视连接压缩：

+   要监视客户端连接中使用的压缩情况，请使用`Compression_algorithm`和`Compression_level`状态变量。对于当前连接，它们的值分别表示压缩算法和压缩级别。

+   要确定服务器配置允许哪些压缩算法用于传入连接，请检查`protocol_compression_algorithms`系统变量。

+   对于源/复制复制连接，配置的压缩算法和压缩级别可从多个来源获取：

    +   性能模式`replication_connection_configuration`表具有`COMPRESSION_ALGORITHMS`和`ZSTD_COMPRESSION_LEVEL`列。

    +   `mysql.slave_master_info`系统表具有`Master_compression_algorithms`和`Master_zstd_compression_level`列。如果存在`master.info`文件，则其中也包含这些值的行。
