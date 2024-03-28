# 2.8.6 配置 SSL 库支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-ssl-library-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/source-ssl-library-configuration.html)

支持加密连接需要 SSL 库，用于随机数生成的熵，以及其他与加密相关的操作。

如果你从源代码分发中编译 MySQL，**CMake** 默认配置分发以使用已安装的 OpenSSL 库。

要使用 OpenSSL 进行编译，请按照以下步骤：

1.  确保你的系统上安装了 OpenSSL 1.0.1 或更新版本。如果安装的 OpenSSL 版本旧于 1.0.1，**CMake** 在 MySQL 配置时会产生错误。如果需要获取 OpenSSL，请访问 [`www.openssl.org`](http://www.openssl.org)。

1.  `WITH_SSL` **CMake** 选项确定用于编译 MySQL 的 SSL 库（参见 Section 2.8.7, “MySQL Source-Configuration Options”）。默认值是 `-DWITH_SSL=system`，使用 OpenSSL。要明确指定此选项，请指定该选项。例如：

    ```sql
    cmake . -DWITH_SSL=system
    ```

    该命令配置分发以使用已安装的 OpenSSL 库。或者，要明确指定 OpenSSL 安装路径，请使用以下语法。如果你安装了多个版本的 OpenSSL，可以防止 **CMake** 选择错误的版本：

    ```sql
    cmake . -DWITH_SSL=*path_name*
    ```

    从 MySQL 8.0.30 开始，支持替代 OpenSSL 系统包，可以在 EL7 上使用 `WITH_SSL=openssl11` 或在 EL8 上使用 `WITH_SSL=openssl3`。由于这些替代版本的 OpenSSL 不支持 LDAP 和 Kerberos 等认证插件，因此这些插件被禁用。

1.  编译并安装分发。

要检查 **mysqld** 服务器是否支持加密连接，请检查 `have_ssl` 系统变量的值：

```sql
mysql> SHOW VARIABLES LIKE 'have_ssl';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| have_ssl      | YES   |
+---------------+-------+
```

如果值为 `YES`，则服务器支持加密连接。如果值为 `DISABLED`，则服务器能够支持加密连接，但未使用适当的 `--ssl-*`xxx`*` 选项启动以启用加密连接；请参阅 Section 8.3.1, “Configuring MySQL to Use Encrypted Connections”。
