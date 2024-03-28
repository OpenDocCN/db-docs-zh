# 8.8 FIPS 支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fips-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/fips-mode.html)

MySQL 支持 FIPS 模式，如果使用 OpenSSL 3.0 或 OpenSSL 1.0.2 进行编译，并且在运行时可用 OpenSSL 库和 FIPS 对象模块。

服务器端的 FIPS 模式适用于服务器执行的加密操作。这包括复制（源/副本和组复制）和运行在服务器内部的 X 插件。FIPS 模式还适用于客户端尝试连接到服务器。

以下各节描述了 FIPS 模式以及如何在 MySQL 中利用它：

+   FIPS 概述

+   MySQL 中 FIPS 模式的系统要求

+   在 MySQL 中配置 FIPS 模式

### FIPS 概述

联邦信息处理标准 140-2（FIPS 140-2）描述了一种安全标准，联邦（美国政府）机构可能要求用于保护敏感或有价值信息的加密模块。要被视为适用于此类联邦用途，加密模块必须获得 FIPS 140-2 认证。如果旨在保护敏感数据的系统缺乏适当的 FIPS 140-2 证书，联邦机构将无法购买该系统。

诸如 OpenSSL 之类的产品可以在 FIPS 模式下使用，尽管 OpenSSL 库本身未经 FIPS 验证。相反，OpenSSL 库与 OpenSSL FIPS 对象模块一起使用，以使基于 OpenSSL 的应用程序能够在 FIPS 模式下运行。

对于 FIPS 及其在 OpenSSL 中的实现的一般信息，这些参考资料可能会有所帮助：

+   [国家标准与技术研究所 FIPS PUB 140-2](https://doi.org/10.6028/NIST.FIPS.140-2)

+   [OpenSSL FIPS 140-2 安全策略](https://csrc.nist.gov/csrc/media/projects/cryptographic-module-validation-program/documents/security-policies/140sp1747.pdf)

+   [fips_module 手册页面](https://www.openssl.org/docs/man3.0/man7/fips_module.html)

重要

FIPS 模式对加密操作施加条件，例如对可接受的加密算法的限制或对更长密钥长度的要求。对于 OpenSSL，确切的 FIPS 行为取决于 OpenSSL 版本。

### MySQL 中 FIPS 模式的系统要求

要使 MySQL 支持 FIPS 模式，必须满足以下系统要求：

+   在构建时，MySQL 必须使用 OpenSSL 进行编译。如果编译使用与 OpenSSL 不同的 SSL 库，则无法在 MySQL 中使用 FIPS 模式。

    此外，MySQL 必须使用经过 FIPS 认证的 OpenSSL 版本进行编译。OpenSSL 1.0.2 和 OpenSSL 3.0 已经获得认证，但 OpenSSL 1.1.1 没有。最近版本的 MySQL 的二进制发行版在某些平台上使用 OpenSSL 3.0.9 编译，这意味着它们未经 FIPS 认证。这会导致根据系统和 MySQL 配置的不同而产生可用 MySQL 功能的权衡：

    +   使用具有 OpenSSL 1.0.2 和所需的 FIPS 对象模块的系统。在这种情况下，如果您使用使用 OpenSSL 1.0.2 编译的二进制发行版，或者使用 OpenSSL 1.0.2 从源代码编译 MySQL，则可以为 MySQL 启用 FIPS 模式。但是，在这种情况下，您不能使用需要 OpenSSL 1.1.1 的 TLSv1.3 协议或密码套件。此外，您正在使用于 2019 年底达到生命周期终点的 OpenSSL 版本。

    +   使用具有 OpenSSL 1.1.1 或更高版本的系统。在这种情况下，您可以使用二进制包安装 MySQL，并且可以使用 TLSv1.3 协议和密码套件，以及其他已支持的 TLS 协议。但是，您不能为 MySQL 启用 FIPS 模式。

    +   使用具有 OpenSSL 3.0 和所需的 FIPS 对象模块的系统。在这种情况下，如果您使用使用 OpenSSL 3.0 编译的二进制发行版，或者使用 OpenSSL 3.0 从源代码编译 MySQL，则可以为 MySQL 启用 FIPS 模式。可以直接通过 OpenSSL 3.0 配置文件处理 FIPS 模式，而不是使用服务器端系统变量和客户端选项（自 MySQL 8.0.34 起已弃用）。当使用 OpenSSL 3.0 编译 MySQL，并且在运行时可用 OpenSSL 库和 FIPS 对象模块时，服务器会读取 OpenSSL 配置文件，并尊重设置使用 FIPS 提供程序的首选项。

        有关升级到 OpenSSL 3.0 的一般信息，请参阅 [OpenSSL 3.0 迁移指南](https://www.openssl.org/docs/man3.0/man7/migration_guide.html)。

+   在运行时，必须将 OpenSSL 库和 OpenSSL FIPS 对象模块作为共享（动态链接）对象可用。可以构建静态链接的 OpenSSL 对象，但 MySQL 无法使用它们。

FIPS 模式已在 EL7 上对 MySQL 进行了测试，但可能也适用于其他系统。

如果您的平台或操作系统提供 OpenSSL FIPS 对象模块，则可以使用它。否则，您可以从源代码构建 OpenSSL 库和 FIPS 对象模块。请参阅 `fips_module` 手册中的说明（参见 FIPS 概述）。

### 在 MySQL 中配置 FIPS 模式

注意

从 MySQL 8.0.34 开始，本节中描述的服务器端和客户端选项已弃用。

MySQL 在服务器端和客户端启用 FIPS 模式的控制：

+   `ssl_fips_mode` 系统变量控制服务器是否在 FIPS 模式下运行。

+   `--ssl-fips-mode`客户端选项控制特定 MySQL 客户端是否在 FIPS 模式下运行。

`ssl_fips_mode`系统变量和`--ssl-fips-mode`客户端选项允许这些值：

+   `OFF`：禁用 FIPS 模式。

+   `ON`：启用 FIPS 模式。

+   `STRICT`：启用“严格” FIPS 模式。

在服务器端，数值型`ssl_fips_mode`的值为 0、1 和 2 分别等同于`OFF`、`ON`和`STRICT`。

重要

一般来说，`STRICT`比`ON`施加更多限制，但 MySQL 本身除了向 OpenSSL 指定 FIPS 模式值外，没有 FIPS 特定的代码。对于`ON`或`STRICT`的 FIPS 模式的确切行为取决于 OpenSSL 版本。有关详细信息，请参考`fips_module`手册页（参见 FIPS 概述）。

注意

如果 OpenSSL FIPS 对象模块不可用，则`ssl_fips_mode`和`--ssl-fips-mode`的唯一允许值为`OFF`。尝试将 FIPS 模式设置为其他值会导致错误。

服务器端的 FIPS 模式适用于服务器执行的加密操作。这包括复制（源/副本和组复制）和运行在服务器内部的 X 插件。

FIPS 模式也适用于客户端尝试连接到服务器的情况。当启用时，在客户端或服务器端，它限制了可以选择的支持加密密码。然而，启用 FIPS 模式并不要求必须使用加密连接，或者必须加密用户凭据。例如，如果启用了 FIPS 模式，就需要更强的加密算法。特别是，MD5 被限制，因此尝试使用像`RC4-MD5`这样的加密密码建立加密连接是行不通的。但是，FIPS 模式并不阻止建立非加密连接。 （为此，您可以为特定用户帐户使用`CREATE USER`或`ALTER USER`的`REQUIRE`子句，或设置`require_secure_transport`系统变量以影响所有帐户。）
