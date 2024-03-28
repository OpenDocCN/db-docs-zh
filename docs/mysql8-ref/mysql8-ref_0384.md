# 8.3 使用加密连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/encrypted-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/encrypted-connections.html)

8.3.1 配置 MySQL 使用加密连接

8.3.2 加密连接 TLS 协议和密码

8.3.3 创建 SSL 和 RSA 证书和密钥

8.3.4 通过 SSH 从 Windows 远程连接到 MySQL

8.3.5 重用 SSL 会话

在 MySQL 客户端和服务器之间的未加密连接中，可以访问网络的人可以监视所有流量并检查在客户端和服务器之间发送或接收的数据。

当您必须以安全方式在网络上传输信息时，不接受未加密的连接。要使任何类型的数据不可读，请使用加密。加密算法必须包含安全元素，以抵抗许多已知攻击，例如更改加密消息的顺序或重复数据两次。

MySQL 支持使用 TLS（传输层安全性）协议在客户端和服务器之间进行加密连接。TLS 有时被称为 SSL（安全套接字层），但 MySQL 实际上不使用 SSL 协议进行加密连接，因为其加密较弱（参见 第 8.3.2 节，“加密连接 TLS 协议和密码”）。

TLS 使用加密算法确保可以信任通过公共网络接收的数据。它具有检测数据更改、丢失或重放的机制。TLS 还包含使用 X.509 标准进行身份验证的算法。

X.509 使得在互联网上识别某人成为可能。简单来说，应该有一个名为“证书颁发机构”（CA）的实体，为需要的任何人分配电子证书。证书依赖于具有两个加密密钥（公钥和秘密密钥）的非对称加密算法。证书所有者可以向另一方展示证书作为身份证明。证书由其所有者的公钥组成。使用此公钥加密的任何数据只能使用相应的秘密密钥解密，该密钥由证书所有者持有。

MySQL 支持使用 OpenSSL 提供加密连接。有关 OpenSSL 支持的加密协议和密码，请参阅 第 8.3.2 节，“加密连接 TLS 协议和密码”。

默认情况下，MySQL 实例在运行时链接到一个已安装的 OpenSSL 库，以支持加密连接和其他与加密相关的操作。您可以从源代码编译 MySQL，并使用`WITH_SSL` **CMake** 选项指定特定已安装的 OpenSSL 版本的路径或替代的 OpenSSL 系统包。在这种情况下，MySQL 会选择该版本。有关如何执行此操作的说明，请参见 Section 2.8.6, “Configuring SSL Library Support”。

从 MySQL 8.0.11 到 8.0.17，可以使用 wolfSSL 编译 MySQL 作为 OpenSSL 的替代方案。从 MySQL 8.0.18 开始，不再支持 wolfSSL，所有 MySQL 构建都使用 OpenSSL。

您可以使用`Tls_library_version` 系统状态变量在运行时检查 OpenSSL 库的版本，该变量从 MySQL 8.0.30 开始可用。

如果您使用一个版本的 OpenSSL 编译 MySQL，并希望在不重新编译的情况下更改为另一个版本，可以通过编辑动态库加载器路径（Unix 系统上的 `LD_LIBRARY_PATH` 或 Windows 系统上的 `PATH`）来实现。删除编译版本的 OpenSSL 路径，并添加替换版本的路径，在路径上任何其他 OpenSSL 库之前放置它。在启动时，当 MySQL 无法在路径上找到使用`WITH_SSL` 指定的 OpenSSL 版本时，它会使用路径上指定的第一个版本。

默认情况下，MySQL 程序尝试使用加密进行连接，如果服务器支持加密连接，则在无法建立加密连接时回退到未加密连接。有关影响加密连接使用的选项的信息，请参见 Section 8.3.1, “Configuring MySQL to Use Encrypted Connections” 和 Command Options for Encrypted Connections。

MySQL 按连接进行加密，并且对于给定用户的加密使用可以是可选的或强制的。这使您可以根据个别应用程序的要求选择加密或未加密连接。有关如何要求用户使用加密连接的信息，请参见 Section 15.7.1.3, “CREATE USER Statement” 中 `CREATE USER` 语句的 `REQUIRE` 子句的讨论。另请参阅 Section 7.1.8, “Server System Variables” 中 `require_secure_transport` 系统变量的描述。

在源服务器和副本服务器之间可以使用加密连接。参见第 19.3.1 节，“设置复制使用加密连接”。

有关在 MySQL C API 中使用加密连接的信息，请参见支持加密连接。

也可以在 SSH 连接到 MySQL 服务器主机时使用加密连接。例如，请参见第 8.3.4 节，“在 Windows 上通过 SSH 远程连接到 MySQL”。
