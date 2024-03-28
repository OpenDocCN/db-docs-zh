# 32.2 MySQL 企业安全概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-security.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-security.html)

MySQL 企业版提供实现安全功能的插件，使用外部服务：

+   MySQL 企业版包括一个认证插件，使 MySQL 服务器能够使用 LDAP（轻量级目录访问协议）来认证 MySQL 用户。LDAP 认证支持用户名和密码、SASL 以及 GSSAPI/Kerberos 认证方法连接到 LDAP 服务。更多信息，请参见 Section 8.4.1.7，“LDAP 可插拔认证”。

+   MySQL 企业版包括一个认证插件，使 MySQL 服务器能够使用本地 Kerberos 来认证 MySQL 用户使用其 Kerberos Principals。更多信息，请参见 Section 8.4.1.8，“Kerberos 可插拔认证”。

+   MySQL 企业版包括一个认证插件，使 MySQL 服务器能够使用 PAM（可插拔认证模块）来认证 MySQL 用户。PAM 使系统能够使用标准接口访问各种认证方法，如 Unix 密码或 LDAP 目录。更多信息，请参见 Section 8.4.1.5，“PAM 可插拔认证”。

+   MySQL 企业版包括一个在 Windows 上执行外部认证的认证插件，使 MySQL 服务器能够使用本机 Windows 服务来认证客户端连接。已登录 Windows 的用户可以根据其环境中的信息从 MySQL 客户端程序连接到服务器，而无需指定额外密码。更多信息，请参见 Section 8.4.1.6，“Windows 可插拔认证”。

+   MySQL 企业版包括一套掩码和去识别函数，执行子集、随机生成和字典替换以去识别字符串、数字、电话号码、电子邮件等。这些函数使得可以使用多种方法对现有数据进行掩码，如混淆（删除识别特征）、生成格式化的随机数据以及数据替换或替代。更多信息，请参见 Section 8.5，“MySQL 企业数据掩码和去识别”。

+   MySQL 企业版包括一组基于 OpenSSL 库的加密函数，将 OpenSSL 功能暴露在 SQL 级别。更多信息，请参见 Section 32.3，“MySQL 企业加密概述”。

+   MySQL 企业版 5.7 及更高版本包括一个使用 Oracle Key Vault 作为密钥环存储后端的密钥环插件。更多信息，请参见第 8.4.4 节，“MySQL 密钥环”。

+   MySQL 透明数据加密（TDE）为 MySQL 服务器中可能包含敏感数据的所有文件提供静态加密。更多信息，请参见第 17.13 节，“InnoDB 静态数据加密”，第 19.3.2 节，“加密二进制日志文件和中继日志文件”，以及加密审计日志文件。

对于其他相关的企业安全功能，请参见第 32.3 节，“MySQL 企业加密概述”。
