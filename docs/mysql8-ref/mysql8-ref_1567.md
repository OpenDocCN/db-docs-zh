# 22.1 MySQL 文档存储接口

> 原文：[`dev.mysql.com/doc/refman/8.0/en/document-store-interfaces.html`](https://dev.mysql.com/doc/refman/8.0/en/document-store-interfaces.html)

要将 MySQL 作为文档存储使用，您需要使用专用组件和支持与 MySQL 服务器通信的客户端来开发基于文档的应用程序。

+   以下 MySQL 产品支持 X 协议，并允许您在选择的语言中使用 X DevAPI 开发与作为文档存储的 MySQL 服务器通信的应用程序：

    +   MySQL Shell（提供 JavaScript 和 Python 中 X DevAPI 的实现）

    +   Connector/C++

    +   Connector/J

    +   Connector/Node.js

    +   Connector/NET

    +   Connector/Python

+   MySQL Shell 是一个交互式界面，支持 JavaScript、Python 或 SQL 模式的 MySQL。您可以使用 MySQL Shell 来原型应用程序，执行查询和更新数据。安装 MySQL Shell 包含下载和安装 MySQL Shell 的说明。

+   本章中的快速入门指南（教程）将帮助您开始使用 MySQL Shell 与 MySQL 作为文档存储。

    JavaScript 的快速入门指南在这里：第 22.3 节，“JavaScript 快速入门指南：用于文档存储的 MySQL Shell”。

    Python 的快速入门指南在这里：第 22.4 节，“Python 快速入门指南：用于文档存储的 MySQL Shell”。

+   *MySQL Shell 用户指南*在 MySQL Shell 8.0 提供了关于配置和使用 MySQL Shell 的详细信息。
