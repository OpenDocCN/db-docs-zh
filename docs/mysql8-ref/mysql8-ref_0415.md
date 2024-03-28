# 8.4.4 MySQL 密钥环

> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring.html)

8.4.4.1 密钥环组件与密钥环插件的比较

8.4.4.2 密钥环组件安装

8.4.4.3 密钥环插件安装

8.4.4.4 使用 component_keyring_file 基于文件的密钥环组件

8.4.4.5 使用 component_keyring_encrypted_file 加密文件密钥环组件

8.4.4.6 使用 keyring_file 基于文件的密钥环插件

8.4.4.7 使用 keyring_encrypted_file 加密文件密钥环插件

8.4.4.8 使用 keyring_okv KMIP 插件

8.4.4.9 使用 keyring_aws 亚马逊 Web 服务密钥环插件

8.4.4.10 使用 HashiCorp Vault 密钥环插件

8.4.4.11 使用 Oracle Cloud 基础设施 Vault 密钥环组件

8.4.4.12 使用 Oracle Cloud 基础设施 Vault 密钥环插件

8.4.4.13 支持的密钥环密钥类型和长度

8.4.4.14 迁移密钥

8.4.4.15 通用密钥环密钥管理功能

8.4.4.16 插件特定的密钥环密钥管理功能

8.4.4.17 密钥环元数据

8.4.4.18 密钥环命令选项

8.4.4.19 密钥环系统变量

MySQL 服务器支持一个密钥环，使内部服务器组件和插件能够安全地存储敏感信息以供以后检索。该实现包括以下元素：

+   管理支持存储或与存储后端通信的密钥环组件和插件。密钥环的使用涉及从可用组件和插件中安装一个。密钥环组件和插件都管理密钥环数据，但配置不同，可能存在操作上的差异（参见第 8.4.4.1 节，“密钥环组件与密钥环插件”）。

    这些密钥环组件可用：

    +   `component_keyring_file`: 将密钥环数据存储在服务器主机的本地文件中。从 MySQL 8.0.24 版本开始，可在 MySQL 社区版和 MySQL 企业版发行版中使用。参见第 8.4.4.4 节，“使用 component_keyring_file 基于文件的密钥环组件”。

    +   `component_keyring_encrypted_file`: 将密钥环数据存储在服务器主机本地的加密、受密码保护的文件中。自 MySQL 8.0.24 起在 MySQL 企业版发行版中可用。参见 Section 8.4.4.5, “使用 component_keyring_encrypted_file 加密文件型密钥环组件”。

    +   `component_keyring_oci`：将密钥环数据存储在 Oracle Cloud 基础设施 Vault 中。自 MySQL 8.0.31 起在 MySQL 企业版发行版中可用。参见 Section 8.4.4.11, “使用 Oracle Cloud 基础设施 Vault 密钥环组件”。

    可用的密钥环插件包括：

    +   `keyring_file`（自 MySQL 8.0.34 起弃用）：将密钥环数据存储在服务器主机本地的文件中。在 MySQL 社区版和 MySQL 企业版发行版中可用。参见 Section 8.4.4.6, “使用 keyring_file 文件型密钥环插件”。

    +   `keyring_encrypted_file`（自 MySQL 8.0.34 起弃用）：将密钥环数据存储在服务器主机本地的加密、受密码保护的文件中。在 MySQL 企业版发行版中可用。参见 Section 8.4.4.7, “使用 keyring_encrypted_file 加密文件型密钥环插件”。

    +   `keyring_okv`：用于与支持 KMIP 的后端密钥环存储产品（如 Oracle Key Vault 和 Gemalto SafeNet KeySecure Appliance）一起使用的 KMIP 1.1 插件。在 MySQL 企业版发行版中可用。参见 Section 8.4.4.8, “使用 keyring_okv KMIP 插件”。

    +   `keyring_aws`：与亚马逊 Web 服务密钥管理服务通信，用于密钥生成并使用本地文件进行密钥存储。在 MySQL 企业版发行版中可用。参见 Section 8.4.4.9, “使用 keyring_aws 亚马逊 Web 服务密钥环插件”。

    +   `keyring_hashicorp`：与 HashiCorp Vault 通信，用于后端存储。自 MySQL 8.0.18 起在 MySQL 企业版发行版中可用。参见 Section 8.4.4.10, “使用 HashiCorp Vault 密钥环插件”。

    +   `keyring_oci`（自 MySQL 8.0.31 起弃用）：与 Oracle Cloud 基础设施 Vault 通信，用于后端存储。自 MySQL 8.0.22 起在 MySQL 企业版发行版中可用。参见 Section 8.4.4.12, “使用 Oracle Cloud 基础设施 Vault 密钥环插件”。

+   用于 keyring 密钥管理的 keyring 服务接口。此服务可在两个级别访问：

    +   SQL 接口：在 SQL 语句中，调用 Section 8.4.4.15, “General-Purpose Keyring Key-Management Functions” 中描述的函数。

    +   C 接口：在 C 语言代码中，调用 Section 7.6.9.2, “The Keyring Service” 中描述的 keyring 服务函数。

+   密钥元数据访问：

    +   Performance Schema `keyring_keys` 表公开了 keyring 中密钥的元数据。密钥元数据包括密钥 ID、密钥所有者和后端密钥 ID。`keyring_keys` 表不公开任何敏感的 keyring 数据，如密钥内容。自 MySQL 8.0.16 版本起可用。参见 Section 29.12.18.2, “The keyring_keys table”。

    +   Performance Schema `keyring_component_status` 表提供了关于正在使用的 keyring 组件的状态信息，如果已安装。自 MySQL 8.0.24 版本起可用。参见 Section 29.12.18.1, “The keyring_component_status Table”。

+   密钥迁移功能。MySQL 支持在密钥库之间迁移密钥，使得 DBA 可以将 MySQL 安装从一个密钥库切换到另一个。参见 Section 8.4.4.14, “Migrating Keys Between Keyring Keystores”。

+   从 MySQL 8.0.24 版本开始，keyring 插件的实现已经修订为使用组件基础架构。这是通过使用名为 `daemon_keyring_proxy_plugin` 的内置插件来实现的，该插件充当插件和组件服务 API 之间的桥梁。参见 Section 7.6.8, “The Keyring Proxy Bridge Plugin”。

警告

对于加密密钥管理，`component_keyring_file` 和 `component_keyring_encrypted_file` 组件，以及 `keyring_file` 和 `keyring_encrypted_file` 插件并不是用作合规性解决方案。诸如 PCI、FIPS 等安全标准要求使用密钥管理系统来在密钥库或硬件安全模块（HSM）中安全、管理和保护加密密钥。

在 MySQL 中，keyring 服务的消费者包括：

+   `InnoDB` 存储引擎使用 keyring 存储其表空间加密的密钥。参见 Section 17.13, “InnoDB Data-at-Rest Encryption”。

+   MySQL 企业审计使用密钥环存储审计日志文件加密密码。请参见加密审计日志文件。

+   二进制日志和中继日志管理支持基于密钥环的日志文件加密。启用日志文件加密后，密钥环存储用于加密二进制日志文件和中继日志文件密码的密钥。请参见第 19.3.2 节，“加密二进制日志文件和中继日志文件”。

+   解密持久化敏感系统变量的文件密钥的主密钥存储在密钥环中。MySQL 服务器实例必须启用密钥环组件以支持对持久化系统变量值进行安全存储，而不是使用不支持该功能的密钥环插件。请参见持久化敏感系统变量。

有关一般密钥环安装说明，请参见第 8.4.4.2 节，“密钥环组件安装”和第 8.4.4.3 节，“密钥环插件安装”。有关特定密钥环组件或插件的安装和配置信息，请参见描述该组件的部分。

有关使用密钥环函数的信息，请参见第 8.4.4.15 节，“通用密钥环密钥管理函数”。

密钥环组件、插件和函数访问提供与密钥环接口的密钥环服务。有关访问此服务和编写密钥环插件的信息，请参见第 7.6.9.2 节，“密钥环服务”和编写密钥环插件。
