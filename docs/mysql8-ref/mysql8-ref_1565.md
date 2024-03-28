# 第二十一章 MySQL Shell

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-shell-userguide.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-shell-userguide.html)

MySQL Shell 是用于 MySQL 服务器的高级客户端和代码编辑器。除了提供的 SQL 功能外，类似于**mysql**，MySQL Shell 还提供了 JavaScript 和 Python 的脚本功能，并包括用于与 MySQL 交互的 API。MySQL Shell 是一个可以单独安装的组件。

以下讨论简要描述了 MySQL Shell 的功能。有关更多信息，请参阅 MySQL Shell 手册，网址为`dev.mysql.com/doc/mysql-shell/en/`。

MySQL Shell 包括以下用 JavaScript 和 Python 实现的 API，您可以使用这些 API 开发与 MySQL 交互的代码。

+   当 MySQL Shell 连接到 MySQL 服务器使用 X 协议时，X DevAPI 使开发人员能够同时处理关系型和文档数据。这使您可以将 MySQL 用作文档存储，有时被称为“使用 NoSQL”。有关更多信息，请参阅第二十二章，*将 MySQL 用作文档存储*。有关 X DevAPI 的概念和用法的文档，请参阅《X DevAPI 用户指南》。

+   AdminAPI 使数据库管理员能够使用 InnoDB Cluster，为基于 InnoDB 的 MySQL 数据库提供了高可用性和可伸缩性的集成解决方案，而无需高级 MySQL 专业知识。AdminAPI 还包括对 InnoDB ReplicaSet 的支持，它使您能够以类似于 InnoDB Cluster 的方式管理运行异步 GTID 基于复制的一组 MySQL 实例。此外，AdminAPI 使 MySQL Router 的管理更加简单，包括与 InnoDB Cluster 和 InnoDB ReplicaSet 的集成。请参阅 MySQL AdminAPI。

MySQL Shell 有两个版本，社区版和商业版。社区版免费提供。商业版提供额外的企业功能，成本低廉。
