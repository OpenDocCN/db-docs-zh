# 7.8.2 在 Windows 上运行多个 MySQL 实例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-windows-servers.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-windows-servers.html)

7.8.2.1 在 Windows 命令行上启动多个 MySQL 实例

7.8.2.2 启动多个 MySQL 实例作为 Windows 服务

您可以通过从命令行手动启动每个具有适当操作参数的服务器，或者通过将多个服务器安装为 Windows 服务并以此方式运行它们，在 Windows 上运行多个服务器。有关从命令行或作为服务运行 MySQL 的一般说明，请参见第 2.3 节，“在 Microsoft Windows 上安装 MySQL”。以下各节描述了如何为每个服务器启动具有不同值的那些必须对每个服务器唯一的选项，例如数据目录。这些选项在第 7.8 节，“在一台机器上运行多个 MySQL 实例”中列出。
