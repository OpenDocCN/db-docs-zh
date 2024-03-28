> 原文：[`dev.mysql.com/doc/refman/8.0/en/cleartext-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/cleartext-pluggable-authentication.html)

#### 8.4.1.4 客户端端明文可插拔身份验证

有一个客户端身份验证插件可用，使客户端可以将密码以明文形式发送到服务器，而无需进行哈希或加密。此插件内置于 MySQL 客户端库中。

以下表格显示了插件名称。

**Table 8.19 Plugin and Library Names for Cleartext Authentication**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | 无，请参见讨论 |
| 客户端插件 | `mysql_clear_password` |
| 库文件 | 无（插件内置） |

许多客户端身份验证插件在客户端将密码发送到服务器之前对密码进行哈希或加密。这使客户端可以避免以明文形式发送密码。

对于需要服务器接收客户端输入的密码的身份验证方案，无法进行哈希或加密。在这种情况下，将使用客户端端的`mysql_clear_password`插件，该插件使客户端可以将密码以明文形式发送到服务器。没有相应的服务器端插件。相反，`mysql_clear_password`可以与需要明文密码的任何服务器端插件一起在客户端端使用（例如 PAM 和简单的 LDAP 身份验证插件；请参见 Section 8.4.1.5, “PAM Pluggable Authentication”，以及 Section 8.4.1.7, “LDAP Pluggable Authentication”）。

下面的讨论提供了特定于明文插件身份验证的使用信息。有关 MySQL 中可插拔身份验证的一般信息，请参见 Section 8.2.17, “Pluggable Authentication”。

注意

在某些配置中，以明文形式发送密码可能会带来安全问题。为避免问题，如果存在密码可能被拦截的可能性，客户端应该使用一种保护密码的方法连接到 MySQL 服务器。可能的方法包括 SSL（请参见 Section 8.3, “Using Encrypted Connections”）、IPsec 或私有网络。

为了减少意外使用`mysql_clear_password`插件的可能性，MySQL 客户端必须显式启用它。可以通过几种方式实现：

+   将`LIBMYSQL_ENABLE_CLEARTEXT_PLUGIN`环境变量设置为以`1`、`Y`或`y`开头的值。这将为所有客户端连接启用该插件。

+   **mysql**, **mysqladmin**, **mysqlcheck**, **mysqldump**, **mysqlshow**, 以及 **mysqlslap** 客户端程序支持 `--enable-cleartext-plugin` 选项，可以在每次调用时启用该插件。

+   `mysql_options()` C API 函数支持 `MYSQL_ENABLE_CLEARTEXT_PLUGIN` 选项，可以在每个连接上启用该插件。此外，任何使用 `libmysqlclient` 并读取选项文件的程序都可以通过在客户端库读取的选项组中包含 `enable-cleartext-plugin` 选项来启用该插件。
