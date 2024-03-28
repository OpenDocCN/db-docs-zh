# 6.1 MySQL 程序概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/programs-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/programs-overview.html)

在 MySQL 安装中有许多不同的程序。本节提供了它们的简要概述。后续章节将更详细地描述每个程序，除了 NDB 集群程序。每个程序的描述都指示了其调用语法和支持的选项。第 25.5 节，“NDB 集群程序”描述了专用于 NDB 集群的程序。

大多数 MySQL 发行版都包含所有这些程序，除了那些特定于平台的程序。（例如，服务器启动脚本在 Windows 上不使用。）例外是 RPM 发行版更加专业化。服务器有一个 RPM，另一个是客户端程序，依此类推。如果您似乎缺少一个或多个程序，请参见第二章，“安装 MySQL”，了解发行版类型及其包含内容的信息。可能是您使用的发行版不包含所有程序，您需要安装额外的软件包。

每个 MySQL 程序都有许多不同的选项。大多数程序提供一个`--help`选项，您可以使用它来获取程序不同选项的描述。例如，尝试**mysql --help**。

您可以通过在命令行或选项文件中指定选项来覆盖 MySQL 程序的默认选项值。有关调用程序和指定程序选项的一般信息，请参见第 6.2 节，“使用 MySQL 程序”。

MySQL 服务器，**mysqld**，是 MySQL 安装中大部分工作都由其完成的主要程序。服务器附带几个相关脚本，帮助您启动和停止服务器：

+   **mysqld**

    SQL 守护程序（即 MySQL 服务器）。要使用客户端程序，**mysqld**必须在运行，因为客户端通过连接到服务器来访问数据库。参见第 6.3.1 节，“mysqld — MySQL 服务器”。

+   **mysqld_safe**

    一个服务器启动脚本。**mysqld_safe**尝试启动**mysqld**。参见第 6.3.2 节，“mysqld_safe — MySQL 服务器启动脚本”。

+   **mysql.server**

    一个服务器启动脚本。该脚本用于使用包含启动特定运行级别系统服务的脚本的 System V 风格运行目录的系统。它调用**mysqld_safe**来启动 MySQL 服务器。参见第 6.3.3 节，“mysql.server — MySQL 服务器启动脚本”。

+   **mysqld_multi**

    一个服务器启动脚本，可以启动或停止系统上安装的多个服务器。参见第 6.3.4 节，“mysqld_multi — 管理多个 MySQL 服务器”。

几个程序在 MySQL 安装或升级过程中执行设置操作：

+   **comp_err**

    该程序在 MySQL 构建/安装过程中使用。它从错误源文件编译错误消息文件。参见第 6.4.1 节，“comp_err — 编译 MySQL 错误消息文件”。

+   **mysql_secure_installation**

    该程序使您能够提高 MySQL 安装的安全性。参见第 6.4.2 节，“mysql_secure_installation — 改善 MySQL 安装安全性”。

+   **mysql_ssl_rsa_setup**

    注意

    **mysql_ssl_rsa_setup**在 MySQL 8.0.34 中已弃用。

    该程序创建 SSL 证书和密钥文件以及 RSA 密钥对文件，以支持安全连接，如果这些文件丢失。由**mysql_ssl_rsa_setup**创建的文件可用于使用 SSL 或 RSA 进行安全连接。参见第 6.4.3 节，“mysql_ssl_rsa_setup — 创建 SSL/RSA 文件”。

+   **mysql_tzinfo_to_sql**

    该程序使用主机系统 zoneinfo 数据库的内容（描述时区的文件集）加载`mysql`数据库中的时区表。参见第 6.4.4 节，“mysql_tzinfo_to_sql — 加载时区表”。

+   **mysql_upgrade**

    在 MySQL 8.0.16 之前，此程序用于 MySQL 升级操作之后。它会根据 MySQL 新版本中所做的任何更改更新授权表，并在必要时检查表的不兼容性并修复它们。参见 Section 6.4.5，“mysql_upgrade — 检查和升级 MySQL 表”。

    从 MySQL 8.0.16 开始，MySQL 服务器执行先前由**mysql_upgrade**处理的升级任务（有关详细信息，请参见 Section 3.4，“MySQL 升级过程升级了什么”）。

连接到 MySQL 服务器的 MySQL 客户端程序：

+   **mysql**

    用于交互式输入 SQL 语句或从文件中批处理执行的命令行工具。参见 Section 6.5.1，“mysql — MySQL 命令行客户端”。

+   **mysqladmin**

    执行管理操作的客户端，例如创建或删除数据库、重新加载授权表、将表刷新到磁盘以及重新打开日志文件。也可以使用**mysqladmin**从服务器检索版本、进程和状态信息。参见 Section 6.5.2，“mysqladmin — MySQL 服务器管理程序”。

+   **mysqlcheck**

    一个表维护客户端，用于检查、修复、分析和优化表。参见 Section 6.5.3，“mysqlcheck — 表维护程序”。

+   **mysqldump**

    将 MySQL 数据库转储为 SQL、文本或 XML 文件的客户端。参见 Section 6.5.4，“mysqldump — 数据库备份程序”。

+   **mysqlimport**

    将文本文件导入到各自的表中的客户端，使用`LOAD DATA`。参见 Section 6.5.5，“mysqlimport — 数据导入程序”。

+   **mysqlpump**

    将 MySQL 数据库转储为 SQL 文件的客户端。参见 Section 6.5.6，“mysqlpump — 数据库备份程序”。

+   **mysqlsh**

    MySQL Shell 是用于 MySQL 服务器的高级客户端和代码编辑器。参见 MySQL Shell 8.0。除了提供的 SQL 功能外，类似于**mysql**，MySQL Shell 还提供了 JavaScript 和 Python 的脚本功能，并包括用于与 MySQL 一起工作的 API。X DevAPI 使您能够处理关系型和文档数据，参见第二十二章，“*将 MySQL 用作文档存储*”。AdminAPI 使您能够处理 InnoDB 集群，参见 MySQL AdminAPI。

+   **mysqlshow**

    一个用于显示数据库、表、列和索引信息的客户端。参见第 6.5.7 节，“mysqlshow — 显示数据库、表和列信息”。

+   **mysqlslap**

    一个旨在模拟 MySQL 服务器的客户端负载并报告每个阶段的时间的客户端。它的工作方式就像多个客户端正在访问服务器一样。参见第 6.5.8 节，“mysqlslap — 负载仿真客户端”。

MySQL 管理和实用程序：

+   **innochecksum**

    一个离线的`InnoDB`离线文件校验工具。参见第 6.6.2 节，“innochecksum — 离线 InnoDB 文件校验工具”。

+   **myisam_ftdump**

    一个用于显示`MyISAM`表中全文索引信息的实用程序。参见第 6.6.3 节，“myisam_ftdump — 显示全文索引信息”。

+   **myisamchk**

    一个用于描述、检查、优化和修复`MyISAM`表的实用程序。参见第 6.6.4 节，“myisamchk — MyISAM 表维护实用程序”。

+   **myisamlog**

    一个处理`MyISAM`日志文件内容的实用程序。参见第 6.6.5 节，“myisamlog — 显示 MyISAM 日志文件内容”。

+   **myisampack**

    一个用于压缩`MyISAM`表以生成更小的只读表的实用程序。参见第 6.6.6 节，“myisampack — 生成压缩的只读 MyISAM 表”。

+   **mysql_config_editor**

    一个实用程序，可以让您将身份验证凭据存储在一个名为`.mylogin.cnf`的安全、加密的登录路径文件中。参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

+   **mysql_migrate_keyring**

    一个用于在一个密钥环组件和另一个之间迁移密钥的实用程序。参见第 6.6.8 节，“mysql_migrate_keyring — 密钥环密钥迁移实用程序”。

+   **mysqlbinlog**

    一个用于从二进制日志中读取语句的实用程序。二进制日志文件中包含的执行语句日志可用于帮助从崩溃中恢复。参见第 6.6.9 节，“mysqlbinlog — 用于处理二进制日志文件的实用程序”。

+   **mysqldumpslow**

    一个用于读取和总结慢查询日志内容的实用程序。参见第 6.6.10 节，“mysqldumpslow — 总结慢查询日志文件”。

MySQL 程序开发实用程序：

+   **mysql_config**

    一个生成编译 MySQL 程序所需选项值的 shell 脚本。参见第 6.7.1 节，“mysql_config — 显示编译客户端选项”。

+   **my_print_defaults**

    一个显示选项文件中选项组中存在哪些选项的实用程序。参见第 6.7.2 节，“my_print_defaults — 显示选项文件中的选项”。

杂项实用程序：

+   **lz4_decompress**

    一个解压缩使用 LZ4 压缩创建的**mysqlpump**输出的实用程序。参见第 6.8.1 节，“lz4_decompress — 解压 mysqlpump LZ4 压缩输出”。

+   **perror**

    一个显示系统或 MySQL 错误代码含义的实用程序。参见第 6.8.2 节，“perror — 显示 MySQL 错误消息信息”。

+   **zlib_decompress**

    用于解压缩使用 ZLIB 压缩创建的**mysqlpump**输出的实用程序。请参见第 6.8.3 节“zlib_decompress — 解压缩 mysqlpump ZLIB 压缩输出”。

Oracle 公司还提供了 MySQL Workbench GUI 工具，用于管理 MySQL 服务器和数据库，创建、执行和评估查询，并将模式和数据从其他关系数据库管理系统迁移到 MySQL 中使用。

MySQL 客户端程序使用 MySQL 客户端/服务器库与服务器通信，使用以下环境变量。

| 环境变量 | 含义 |
| --- | --- |
| `MYSQL_UNIX_PORT` | 默认的 Unix 套接字文件；用于连接到`localhost` |
| `MYSQL_TCP_PORT` | 默认端口号；用于 TCP/IP 连接 |
| `MYSQL_DEBUG` | 调试时的调试跟踪选项 |
| `TMPDIR` | 创建临时表和文件的目录 |

要查看 MySQL 程序使用的所有环境变量列表，请参见第 6.9 节“环境变量”。
