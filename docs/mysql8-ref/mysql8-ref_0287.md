# 7.6 MySQL 服务器插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-plugins.html`](https://dev.mysql.com/doc/refman/8.0/en/server-plugins.html)

7.6.1 安装和卸载插件

7.6.2 获取服务器插件信息

7.6.3 MySQL 企业线程池

7.6.4 Rewriter Query Rewrite 插件

7.6.5 ddl_rewriter 插件

7.6.6 版本标记

7.6.7 Clone 插件

7.6.8 Keyring Proxy Bridge 插件

7.6.9 MySQL 插件服务

MySQL 支持一个插件 API，可以创建服务器插件。插件可以在服务器启动时加载，也可以在运行时加载和卸载而无需重新启动服务器。此接口支持的插件包括但不限于存储引擎、`INFORMATION_SCHEMA`表、全文解析器插件和服务器扩展。

MySQL 发行版包括几个实现服务器扩展的插件：

+   用于验证客户端连接到 MySQL 服务器的尝试的插件。有几种认证协议的插件可用。参见第 8.2.17 节，“可插拔认证”。

+   一个连接控制插件，使管理员能够在一定数量的连续失败的客户端连接尝试之后引入逐渐增加的延迟。参见第 8.4.2 节，“连接控制插件”。

+   一个密码验证插件实现密码强度策略并评估潜在密码的强度。参见第 8.4.3 节，“密码验证组件”。

+   半同步复制插件实现了一个接口，允许源继续进行，只要至少有一个副本对每个事务做出响应。参见第 19.4.10 节，“半同步复制”。

+   Group Replication 使您能够在一组 MySQL 服务器实例之间创建高可用的分布式 MySQL 服务，具有数据一致性、冲突检测和解决以及组成员服务等功能。参见第二十章，*Group Replication*。

+   MySQL 企业版包括一个线程池插件，通过有效管理大量客户端连接的语句执行线程来增加服务器性能。参见第 7.6.3 节，“MySQL 企业线程池”。

+   MySQL Enterprise Edition 包括用于监视和记录连接和查询活动的审计插件。参见第 8.4.5 节，“MySQL 企业审计”。

+   MySQL Enterprise Edition 包括一个防火墙插件，实现应用级防火墙，使数据库管理员可以根据匹配接受的语句模式允许或拒绝 SQL 语句执行。参见第 8.4.7 节，“MySQL 企业防火墙”。

+   查询重写插件检查 MySQL 服务器接收到的语句，并在服务器执行之前可能对其进行重写。参见第 7.6.4 节，“重写器查询重写插件”，以及第 7.6.5 节，“ddl_rewriter 插件”。

+   版本令牌允许创建和同步服务器令牌，应用程序可以使用这些令牌来防止访问不正确或过时的数据。版本令牌基于一个实现`version_tokens`插件和一组可加载函数的插件库。参见第 7.6.6 节，“版本令牌”。

+   Keyring 插件为敏感信息提供安全存储。参见第 8.4.4 节，“MySQL Keyring”。

    在 MySQL 8.0.24 中，MySQL Keyring 开始从插件过渡到使用组件基础架构，通过使用名为`daemon_keyring_proxy_plugin`的插件作为插件和组件服务 API 之间的桥梁来实现。参见第 7.6.8 节，“Keyring 代理桥插件”。

+   X Plugin 扩展了 MySQL Server 的功能，使其能够作为文档存储库。运行 X Plugin 使 MySQL Server 能够使用 X 协议与客户端通信，该协议旨在将 MySQL 的 ACID 兼容存储能力作为文档存储库暴露出来。参见第 22.5 节，“X 插件”。

+   克隆允许从本地或远程 MySQL 服务器实例克隆`InnoDB`数据。参见第 7.6.7 节，“克隆插件”。

+   测试框架插件测试服务器服务。有关这些插件的信息，请参阅 MySQL Server Doxygen 文档中的 Plugins for Testing Plugin Services 部分，网址为`dev.mysql.com/doc/index-other.html`。

以下各节描述了如何安装和卸载插件，以及如何在运行时确定已安装的插件并获取有关它们的信息。有关编写插件的信息，请参见 MySQL 插件 API。
