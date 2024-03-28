# 18.8 **FEDERATED** 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/federated-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/federated-storage-engine.html)

18.8.1 FEDERATED 存储引擎概述

18.8.2 如何创建 FEDERATED 表

18.8.3 FEDERATED 存储引擎注意事项和提示

18.8.4 FEDERATED 存储引擎资源

`FEDERATED`存储引擎允许您访问远程 MySQL 数据库中的数据，而无需使用复制或集群技术。查询本地`FEDERATED`表会自动从远程（联合）表中提取数据。本地表上不存储任何数据。

如果从源代码构建 MySQL 并包含`FEDERATED`存储引擎，请使用`-DWITH_FEDERATED_STORAGE_ENGINE`选项调用**CMake**。

运行服务器默认未启用`FEDERATED`存储引擎；要启用`FEDERATED`，必须使用`--federated`选项启动 MySQL 服务器二进制文件。

要查看`FEDERATED`引擎的源代码，请查看 MySQL 源代码分发中的`storage/federated`目录。
