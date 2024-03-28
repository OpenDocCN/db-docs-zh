# A.9 MySQL 8.0 FAQ：安全性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-security.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-security.html)

A.9.1\. 我在哪里可以找到有关 MySQL 安全问题的文档？](faqs-security.html#faq-mysql-where-docs-security)

A.9.2\. MySQL 8.0 中的默认身份验证插件是什么？](faqs-security.html#faq-mysql-default-authentication-plugin)

A.9.3\. MySQL 8.0 是否原生支持 SSL？](faqs-security.html#faq-mysql-have-native-ssl)

A.9.4\. SSL 支持是否内置于 MySQL 二进制文件中，还是必须重新编译二进制文件才能启用？](faqs-security.html#faq-mysql-is-ssl-available)

A.9.5\. MySQL 8.0 是否内置对 LDAP 目录的身份验证？](faqs-security.html#faq-mysql-have-builtin-ldap)

A.9.6\. MySQL 8.0 是否包含基于角色的访问控制（RBAC）支持？](faqs-security.html#faq-mysql-have-builtin-rbac)

A.9.7\. MySQL 8.0 是否支持 TLS 1.0 和 1.1？](faqs-security.html#faq-mysql-tls-versions)

| **A.9.1.** | 我在哪里可以找到有关 MySQL 安全问题的文档？ |
| --- | --- |

|  | 最好的起点是第八章，*安全*。MySQL 文档的其他部分，您可能会发现与特定安全问题相关的包括以下内容：

+   第 8.1.1 节，“安全准则”.

+   第 8.1.3 节，“使 MySQL 免受攻击者攻击”.

+   第 B.3.3.2 节，“如何重置根密码”.

+   第 8.1.5 节，“如何以普通用户身份运行 MySQL”.

+   第 8.1.4 节，“与安全相关的 mysqld 选项和变量”.

+   第 8.1.6 节，“LOAD DATA LOCAL 的安全注意事项”.

+   第 2.9 节，“安装后设置和测试”.

+   第 8.3 节，“使用加密连接”.

+   可加载函数安全预防措施.

还有安全部署指南，提供了使用 MySQL 企业版服务器的通用二进制分发功能的部署程序，以管理 MySQL 安装的安全性。 |

| **A.9.2.** | MySQL 8.0 中的默认身份验证插件是什么？ |
| --- | --- |
|  | MySQL 8.0 中默认的身份验证插件是`caching_sha2_password`。有关此插件的信息，请参阅 Section 8.4.1.2, “Caching SHA-2 Pluggable Authentication”。`caching_sha2_password`插件提供比`mysql_native_password`插件更安全的密码加密（前一个 MySQL 系列的默认插件）。有关此默认插件更改对服务器操作和服务器与客户端及连接器兼容性的影响的信息，请参阅 caching_sha2_password as the Preferred Authentication Plugin。有关可插拔身份验证和其他可用身份验证插件的一般信息，请参阅 Section 8.2.17, “Pluggable Authentication”，以及 Section 8.4.1, “Authentication Plugins”。 |
| **A.9.3.** | MySQL 8.0 是否原生支持 SSL？ |
|  | 大多数 8.0 二进制文件支持客户端和服务器之间的 SSL 连接。请参阅 Section 8.3, “Using Encrypted Connections”。您还可以使用 SSH 隧道连接，如果（例如）客户端应用程序不支持 SSL 连接。有关示例，请参阅 Section 8.3.4, “Connecting to MySQL Remotely from Windows with SSH”。 |
| **A.9.4.** | SSL 支持是否内置于 MySQL 二进制文件中，还是必须重新编译二进制文件才能启用？ |
|  | 大多数 8.0 二进制文件已启用 SSL，用于客户端/服务器连接的安全、身份验证或两者兼有。请参阅 Section 8.3, “Using Encrypted Connections”。 |
| **A.9.5.** | MySQL 8.0 是否内置对 LDAP 目录的身份验证？ |
|  | 企业版包括一个 PAM 身份验证插件，支持对 LDAP 目录进行身份验证。 |
| **A.9.6.** | MySQL 8.0 是否支持基于角色的访问控制（RBAC）？ |
|  | 目前不支持。 |
| **A.9.7.** | MySQL 8.0 是否支持 TLS 1.0 和 1.1？ |

|  | 从 MySQL 8.0.28 开始，不再支持 TLSv1 和 TLSv1.1 连接协议。这些协议从 MySQL 8.0.26 开始被弃用。有关该移除的后果，请参阅 移除对 TLSv1 和 TLSv1.1 协议的支持。移除对 TLS 版本 1.0 和 1.1 的支持是因为这些协议版本过时，分别于 1996 年和 2006 年发布。使用的算法是薄弱且过时的。除非您正在使用非常旧的 MySQL Server 或连接器版本，否则不太可能有使用 TLS 1.0 或 1.1 的连接。MySQL 连接器和客户端默认选择可用的最高 TLS 版本。*MySQL Server 何时添加对 TLS 1.2 的支持？* MySQL Community Server 在 2019 年切换到 OpenSSL 时为 MySQL 5.6、5.7 和 8.0 添加了 TLS 1.2 支持。对于 MySQL Enterprise Edition，OpenSSL 在 2015 年为 MySQL Server 5.7.10 添加了 TLS 1.2 支持。*如何查看正在使用的 TLS 版本？* 对于 MySQL 5.7 或 8.0，通过运行此查询查看是否正在使用 TLS 1.0 或 1.1：

```sql
SELECT
  `session_ssl_status`.`thread_id`, `session_ssl_status`.`ssl_version`,
  `session_ssl_status`.`ssl_cipher`, `session_ssl_status`.`ssl_sessions_reused`
FROM `sys`.`session_ssl_status` 
WHERE ssl_version NOT IN ('TLSv1.3','TLSv1.2');
```

如果列出了使用 TLSv1.0 或 TLSv1.1 的线程，您可以通过运行此查询确定此连接来自何处：

```sql
SELECT thd_id,conn_id, user, db, current_statement, program_name 
FROM sys.processlist
WHERE thd_id IN (
                  SELECT `session_ssl_status`.`thread_id`
                  FROM `sys`.`session_ssl_status` 
                  WHERE ssl_version NOT IN ('TLSv1.3','TLSv1.2')
                );
```

或者，您可以运行此查询：

```sql
SELECT * 
FROM sys.session 
WHERE thd_id IN (
                  SELECT `session_ssl_status`.`thread_id`
                  FROM `sys`.`session_ssl_status` 
                  WHERE ssl_version NOT IN ('TLSv1.3','TLSv1.2')
                );
```

这些查询提供了确定哪个应用程序不支持 TLS 1.2 或 1.3 所需的详细信息，并针对升级进行目标升级。*是否有其他测试 TLS 1.0 或 1.1 的选项？* 是的，您可以在将服务器升级到新版本之前禁用这些版本。明确指定要使用的版本，可以在 `mysql.cnf`（或 `mysql.ini`）中或通过 `SET PERSIST` 来指定，例如：`--tls-version=TLSv12`。*所有 MySQL 连接器（5.7 和 8.0）是否支持 TLS 1.2 及更高版本？C 和 C++ 应用程序使用 `libmysql` 呢？* 对于使用社区 `libmysqlclient` 库的 C 和 C++ 应用程序，请使用基于 OpenSSL 的库（即，*不要* 使用 YaSSL）。在 2018 年统一使用 OpenSSL（分别在 MySQL 8.0.4 和 5.7.28 中）。对于 Connector/ODBC 和 Connector/C++ 也是如此。要确定使用了哪些库依赖项，请运行以下命令查看 OpenSSL 是否被列出。在 Linux 上，使用以下命令：

```sql
$> sudo ldd usr/local/mysql/lib/libmysqlclient.a &#124; grep -i openssl
```

在 MacOS 上，使用以下命令：

```sql
$> sudo otool -l /usr/local/mysql/lib/libmysqlclient.a &#124; grep -i openssl
```

*关于 Connector/J？* Java 8 在 2014 年 1 月将 TLS 1.2 作为默认版本；在此之前就已经支持了 TLS 1.2，所以除非你在运行一个非常老的版本的 Connector/J，否则你已经支持 TLS 1.2 了。*关于 Connector/NET？* 对于 .NET 应用程序，微软在 2020 年底停止了对 TLS 1.0 和 1.1 的支持。TLS 1.2 在 2012 年就已经添加了支持。你需要一个非常老的版本的 Connector/NET 才不支持 TLS 1.2。*关于 Connector/Python？* 这取决于你运行的 Python 版本。Python 2.6 中的 SSL 模块只支持到 1.0 版本的 TLS。在这种情况下，你需要升级到 Python 2.7.9 或更高版本，或者使用 Python 3.x，两者都支持更新版本的 TLS。详情请参见 Connector/Python 版本 和 [`www.calazan.com/how-to-check-if-your-python-app-supports-tls-12/`](https://www.calazan.com/how-to-check-if-your-python-app-supports-tls-12/)。*关于 Connector/Node.js 或 Node MySQL2？* TLS 随 `nodejs` 一起提供，所有支持的 Node.js 版本使用 OpenSSL v1.1.1（截至 2020 年 4 月），再次支持 TLS 1.2 及更高版本。*关于 PHP？* [这些版本的 PHP](https://www.php.net/supported-versions.php) 支持 TLS 1.2 及更高版本。
