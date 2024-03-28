# 32.5 MySQL 企业防火墙概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-firewall.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-firewall.html)

MySQL 企业版包括 MySQL 企业防火墙，这是一个应用级防火墙，允许数据库管理员根据匹配接受语句模式的允许列表来允许或拒绝 SQL 语句的执行。这有助于加固 MySQL 服务器，防范诸如 SQL 注入或试图利用应用程序以外的合法查询工作负载特征来攻击的行为。

每个在防火墙中注册的 MySQL 账户都有自己的语句允许列表，可以根据账户进行定制化保护。对于给定的账户，防火墙可以在记录或保护模式下运行，用于训练接受的语句模式或保护不可接受的语句。

更多信息，请参见 第 8.4.7 节，“MySQL 企业防火墙”。
