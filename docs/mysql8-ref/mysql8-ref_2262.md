# 32.6 MySQL 企业版线程池概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-thread-pool.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-thread-pool.html)

MySQL 企业版包括 MySQL 企业版线程池，使用服务器插件实现。MySQL 服务器中的默认线程处理模型使用一个线程来执行每个客户端连接的语句。随着更多客户端连接到服务器并执行语句，整体性能会下降。在 MySQL 企业版中，一个线程池插件提供了一个旨在减少开销并提高性能的替代线程处理模型。该插件实现了一个线程池，通过有效地管理大量客户端连接的语句执行线程来提高服务器性能。

欲了解更多信息，请参阅第 7.6.3 节，“MySQL 企业版线程池”。
