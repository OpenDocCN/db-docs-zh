> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-plugin-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-plugin-installation.html)

#### 8.4.4.3 Keyring Plugin Installation

Keyring 服务的使用者要求安装一个 keyring 组件或插件：

+   若要使用 keyring 插件，请从这里的说明开始。（此外，有关安装插件的一般信��，请参见 Section 7.6.1, “Installing and Uninstalling Plugins”。)

+   若要使用 keyring 组件，请从 Section 8.4.4.2, “Keyring Component Installation”开始。

+   如果您打算与所选的 keyring 组件或插件一起使用 keyring 函数，请在安装该组件或插件后安装函数，使用 Section 8.4.4.15, “General-Purpose Keyring Key-Management Functions”中的说明。

注意

一次只能启用一个 keyring 组件或插件。不支持启用多个 keyring 组件或插件，结果可能不如预期。

如果您需要支持持久化系统变量值的安全存储，而不是支持该功能的 keyring 插件，则必须在 MySQL Server 实例上启用 keyring 组件。请参阅 Persisting Sensitive System Variables。

MySQL 提供以下 keyring 插件选择：

+   `keyring_file`（自 MySQL 8.0.34 起已弃用）：将 keyring 数据存储在服务器主机本地的文件中。在 MySQL Community Edition 和 MySQL Enterprise Edition 发行版中可用。有关安装替代此插件的组件的说明，请参见 Section 8.4.4.2, “Keyring Component Installation”。

+   `keyring_encrypted_file`（自 MySQL 8.0.34 起已弃用）：将 keyring 数据存储在服务器主机本地的加密、受密码保护的文件中。仅在 MySQL Enterprise Edition 发行版中可用。有关安装替代此插件的组件的说明，请参见 Section 8.4.4.2, “Keyring Component Installation”。

+   `keyring_okv`：用于与 KMIP 兼容的后端 keyring 存储产品（如 Oracle Key Vault 和 Gemalto SafeNet KeySecure Appliance）一起使用的 KMIP 1.1 插件。仅在 MySQL Enterprise Edition 发行版中可用。

+   `keyring_aws`：与亚马逊 Web 服务密钥管理服务通信，用于密钥生成的后端，并使用本地文件进行密钥存储。仅在 MySQL Enterprise Edition 发行版中可用。

+   `keyring_hashicorp`：与 HashiCorp Vault 通信，用于后端存储。仅在 MySQL Enterprise Edition 发行版中可用。

+   `keyring_oci`（自 MySQL 8.0.31 起已弃用）：与 Oracle Cloud 基础设施 Vault 进行后端存储通信。请参阅第 8.4.4.12 节，“使用 Oracle Cloud 基础设施 Vault 密钥环插件”。

要让服务器能够使用插件库文件，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如果需要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

密钥环组件或插件必须在服务器启动序列的早期加载，以便其他组件在其自身初始化期间根据需要访问它。例如，`InnoDB` 存储引擎使用密钥环进行表空间加密，因此必须在 `InnoDB` 初始化之前加载并可用密钥环组件或插件。

每个密钥环插件的安装方式类似。以下说明描述了如何安装`keyring_file`。要使用不同的密钥环插件，请将其名称替换为`keyring_file`。

`keyring_file`插件库文件基本名称为`keyring_file`。文件名后缀因平台而异（例如，Unix 和类 Unix 系统为`.so`，Windows 为`.dll`）。

要加载插件，请使用`--early-plugin-load`选项来命名包含插件库文件的插件。例如，在插件库文件后缀为`.so`的平台上，请在服务器`my.cnf`文件中使用以下行，根据需要调整`.so`后缀以适应您的平台：

```sql
[mysqld]
early-plugin-load=keyring_file.so
```

在启动服务器之前，请查看您选择的密钥环插件的注意事项，以获取特定于该插件的配置说明：

+   `keyring_file`: 第 8.4.4.6 节，“使用基于文件的 keyring_file 插件”。

+   `keyring_encrypted_file`: 第 8.4.4.7 节，“使用加密文件的 keyring_encrypted_file 插件”。

+   `keyring_okv`: 第 8.4.4.8 节，“使用 keyring_okv KMIP 插件”。

+   `keyring_aws`: 第 8.4.4.9 节，“使用 keyring_aws 亚马逊网络服务密钥环插件”

+   `keyring_hashicorp`: 第 8.4.4.10 节，“使用 HashiCorp Vault 密钥环插件”

+   `keyring_oci`: 第 8.4.4.12 节，“使用 Oracle Cloud 基础设施 Vault 密钥环插件”

在执行任何特定于插件的配置后，启动服务器。通过检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）来验证插件安装。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'keyring%';
+--------------+---------------+
| PLUGIN_NAME  | PLUGIN_STATUS |
+--------------+---------------+
| keyring_file | ACTIVE        |
+--------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

插件可以通过`--early-plugin-load`之外的方法加载，例如`--plugin-load`或`--plugin-load-add`选项或`INSTALL PLUGIN`语句。然而，使用这些方法加载的密钥环插件可能在服务器启动序列中对于某些使用密钥环的组件来说太晚了，比如`InnoDB`：

+   使用`--plugin-load`或`--plugin-load-add`进行插件加载发生在`InnoDB`初始化之后。

+   使用`INSTALL PLUGIN`安装的插件会在`mysql.plugin`系统表中注册，并在后续服务器重新启动时自动加载。然而，由于`mysql.plugin`是一个`InnoDB`表，其中列出的任何插件只能在`InnoDB`初始化后的启动期间加载。

如果在组件尝试访问密钥环服务时没有可用的密钥环组件或插件，则该服务无法被该组件使用。因此，该组件可能无法初始化或以有限功能初始化。例如，如果`InnoDB`在初始化时发现存在加密表空间，它会尝试访问密钥环。如果密钥环不可用，`InnoDB`只能访问未加密的表空间。为确保`InnoDB`也可以访问加密表空间，请使用`--early-plugin-load`加载密钥环插件。
