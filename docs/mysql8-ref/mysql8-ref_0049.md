> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-select-server.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-select-server.html)

#### 2.3.4.3 选择 MySQL 服务器类型

以下表显示了 MySQL 8.0 中 Windows 可用的服务器。

| 二进制 | 描述 |
| --- | --- |
| **mysqld** | 优化的二进制文件，支持命名管道 |
| **mysqld-debug** | 类似于**mysqld**，但编译时带有完整调试和自动内存分配检查 |

所有前述的二进制文件都针对现代 Intel 处理器进行了优化，但应该在任何 Intel i386 级别或更高级别的处理器上运行。

发行版中的每个服务器都支持相同的存储引擎。`SHOW ENGINES`语句显示给定服务器支持哪些引擎。

所有 Windows MySQL 8.0 服务器都支持数据库目录的符号链接。

MySQL 在所有 Windows 平台上支持 TCP/IP。如果您启动具有`named_pipe`系统变量启用的服务器，则 Windows 上的 MySQL 服务器还支持命名管道。必须显式启用此变量，因为一些用户在使用命名管道时关闭 MySQL 服务器时遇到问题。默认情况下，无论平台如何，都使用 TCP/IP，因为在许多 Windows 配置中，命名管道比 TCP/IP 慢。
