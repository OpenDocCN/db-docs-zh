# 22.4 Python 快速入门指南：MySQL Shell 用于文档存储

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-tutorial-python.html)

22.4.1 MySQL Shell

22.4.2 下载和导入 world_x 数据库

22.4.3 文档和集合

22.4.4 关系表

22.4.5 表中的文档

本快速入门指南提供了使用 MySQL Shell 与文档存储应用程序进行交互原型设计的说明。该指南包括以下主题：

+   MySQL 功能、MySQL Shell 和 `world_x` 示例模式的简介。

+   管理集合和文档的操作。

+   管理关系表的操作。

+   适用于表中文档的操作。

要按照这个快速入门指南，您需要安装了 X 插件的 MySQL 服务器，默认情况下在 8.0 版本中，以及用作客户端的 MySQL Shell。MySQL Shell 包括 X DevAPI，它在 JavaScript 和 Python 中都有实现，使您能够使用 X 协议连接到 MySQL 服务器实例，并将服务器用作文档存储。

### 相关信息

+   MySQL Shell 8.0 提供了有关 MySQL Shell 的更深入信息。

+   有关本快速入门指南中使用的工具的更多信息，请参阅安装 MySQL Shell 和第 22.5 节，“X 插件”。

+   有关 MySQL Shell 支持的语言的更多信息，请参阅支持的语言。

+   X DevAPI 用户指南提供了更多使用 X DevAPI 开发使用 MySQL 作为文档存储的应用程序的示例。

+   还提供了一个 JavaScript 快速入门指南。
