# 7.6.9 MySQL 插件服务

> 原文：[`dev.mysql.com/doc/refman/8.0/en/plugin-services.html`](https://dev.mysql.com/doc/refman/8.0/en/plugin-services.html)

7.6.9.1 锁定服务

7.6.9.2 密钥环服务

MySQL 服务器插件可以访问服务器的“插件服务”。插件服务接口通过暴露插件可以调用的服务器功能来补充插件 API。有关编写插件服务的开发人员信息，请参阅 MySQL 插件服务。以下部分描述了在 SQL 和 C 语言级别可用的插件服务。
