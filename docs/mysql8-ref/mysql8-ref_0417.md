> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-component-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-component-installation.html)

#### 8.4.4.2 密钥环组件安装

密钥环服务使用者要求必须安装密钥环组件或插件：

+   若要使用密钥环组件，请从这里的说明开始。

+   若要使用密钥环插件，请从 Section 8.4.4.3, “Keyring Plugin Installation”开始。

+   如果打算与所选的密钥环组件或插件一起使用密钥环函数，请在安装该组件或插件后安装这些函数，使用 Section 8.4.4.15, “General-Purpose Keyring Key-Management Functions”中的说明。

注意

一次只能启用一个密钥环组件或插件。启用多个密钥环组件或插件是不受支持的，结果可能不如预期。

MySQL 提供以下密钥环组件选择：

+   `component_keyring_file`: 将密钥环数据存储在服务器主机上的文件中。在 MySQL Community Edition 和 MySQL Enterprise Edition 发行版中可用。

+   `component_keyring_encrypted_file`: 将密钥环数据存储在服务器主机上的加密、受密码保护的文件中。在 MySQL Enterprise Edition 发行版中可用。

+   `component_keyring_oci`: 将密钥环数据存储在 Oracle Cloud Infrastructure Vault 中。在 MySQL Enterprise Edition 发行版中可用。

要供服务器使用，组件库文件必须位于 MySQL 插件目录中（由 `plugin_dir` 系统变量命名的目录）。如有必要，在服务器启动时通过设置 `plugin_dir` 的值来配置插件目录位置。

必须在服务器启动序列的早期加载密钥环组件或插件，以便其他组件在它们自己的初始化过程中可以根据需要访问它。例如，`InnoDB` 存储引擎使用密钥环进行表空间加密，因此必须在 `InnoDB` 初始化之前加载并可用密钥环组件或插件。

注意

如果需要支持持久化系统变量值的安全存储，则必须在 MySQL 服务器实例上启用密钥环组件。密钥环插件不支持此功能。请参阅持久化敏感系统变量。

与密钥环插件不同，密钥环组件不是使用`--early-plugin-load`服务器选项加载或使用系统变量配置的。相反，服务器在启动期间使用清单确定要加载哪个密钥环组件，并且加载的组件在初始化时会查阅自己的配置文件。因此，要安装密钥环组件，您必须：

1.  编写一个清单，告诉服务器要加载哪个密钥环组件。

1.  为该密钥环组件编写一个配置文件。

安装密钥环组件的第一步是编写一个指示要加载哪个组件的清单。在启动期间，服务器会读取全局清单文件或与本地清单文件配对的全局清单文件：

+   服务器将尝试从安装服务器的目录中读取其全局清单文件。

+   如果全局清单文件指示使用本地清单文件，则服务器将尝试从数据目录读取其本地清单文件。

+   尽管全局和本地清单文件位于不同的目录中，但文件名在两个位置都是`mysqld.my`。

+   清单文件不存在并不是一个错误。在这种情况下，服务器不会尝试加载与文件相关联的任何组件。

本地清单文件允许为服务器的多个实例设置组件加载，以便每个服务器实例的加载指令特定于给定的数据目录实例。这使得不同的 MySQL 实例可以使用不同的密钥环组件。

服务器清单文件具有以下属性：

+   清单文件必须采用有效的 JSON 格式。

+   清单文件允许这些项：

    +   `"read_local_manifest"`：此项仅允许在全局清单文件中。如果该项不存在，则服务器仅使用全局清单文件。如果该项存在，则其值为`true`或`false`，指示服务器是否应从本地清单文件中读取组件加载信息。

        如果全局清单文件中存在`"read_local_manifest"`项以及其他项，则服务器首先检查`"read_local_manifest"`项的值：

        +   如果值为`false`，服务器将处理全局清单文件中的其他项，并忽略本地清单文件。

        +   如果值为`true`，服务器将忽略全局清单文件中的其他项，并尝试读取本地清单文件。

    +   `"components"`：此项指示要加载的组件。该项值是一个字符串，指定一个有效的组件 URN，例如`"file://component_keyring_file"`。组件 URN 以`file://`开头，并指示位于 MySQL 插件目录中实现组件的库文件的基本名称。

+   服务器对清单文件的访问权限应为只读。例如，`mysqld.my`服务器清单文件可能由`root`拥有，并对`root`读/写，但对用于运行 MySQL 服务器的帐户应为只读。如果在启动期间发现清单文件对该帐户为读/写，则服务器会向错误日志写入警告，建议将文件设置为只读。

+   数据库管理员有责任创建要使用的任何清单文件，并确保它们的访问模式和内容正确。如果发生错误，服务器启动将失败，管理员必须根据服务器错误日志中的诊断信息纠正任何问题。

根据前面的清单文件属性，要配置服务器以加载`component_keyring_file`，请在**mysqld**安装目录中创建一个名为`mysqld.my`的全局清单文件，并在数据目录中可选地创建一个名为`mysqld.my`的本地清单文件。以下说明描述了如何加载`component_keyring_file`。要加载不同的密钥环组件，请用其名称替换`component_keyring_file`。

+   仅使用全局清单文件时，文件内容如下所示：

    ```sql
    {
      "components": "file://component_keyring_file"
    }
    ```

    在**mysqld**安装的目录中创建此文件。

+   或者，要使用全局和本地清单文件对，全局文件如下所示：

    ```sql
    {
      "read_local_manifest": true
    }
    ```

    在**mysqld**安装的目录中创建此文件。

    本地文件如下所示：

    ```sql
    {
      "components": "file://component_keyring_file"
    }
    ```

    在数据目录中创建此文件。

有了清单后，继续配置密钥环组件。要做到这一点，请查看您选择的密钥环组件的说明，以获取特定于该组件的配置说明：

+   `component_keyring_file`: 第 8.4.4.4 节，“使用基于文件的 component_keyring_file 密钥环组件”。

+   `component_keyring_encrypted_file`: 第 8.4.4.5 节，“使用基于文件的 component_keyring_encrypted_file 加密密钥环组件”。

+   `component_keyring_oci`: 第 8.4.4.11 节，“使用 Oracle Cloud 基础设施 Vault 密钥环组件”。

执行任何特定于组件的配置后，启动服务器。通过检查性能模式`keyring_component_status`表来验证组件安装：

```sql
mysql> SELECT * FROM performance_schema.keyring_component_status;
+---------------------+-------------------------------------------------+
| STATUS_KEY          | STATUS_VALUE                                    |
+---------------------+-------------------------------------------------+
| Component_name      | component_keyring_file                          |
| Author              | Oracle Corporation                              |
| License             | GPL                                             |
| Implementation_name | component_keyring_file                          |
| Version             | 1.0                                             |
| Component_status    | Active                                          |
| Data_file           | /usr/local/mysql/keyring/component_keyring_file |
| Read_only           | No                                              |
+---------------------+-------------------------------------------------+
```

`Component_status`值为`Active`表示组件初始化成功。

如果组件无法加载，服务器启动将失败。检查服务器错误日志以获取诊断信息。如果组件加载但由于配置问题而无法初始化，服务器将启动，但`Component_status`值为`Disabled`。检查服务器错误日志，纠正配置问题，并使用`ALTER INSTANCE RELOAD KEYRING`语句重新加载配置。

密钥环组件应该只能通过使用清单文件加载，而不是使用`INSTALL COMPONENT`语句加载。使用该语句加载的密钥环组件可能在服务器启动序列中的某些组件需要使用密钥环时太晚可用，比如`InnoDB`，因为它们在`mysql.component`系统表中注册并在后续服务器重启时自动加载。但`mysql.component`是一个`InnoDB`表，因此在`InnoDB`初始化后才能在启动时加载任何在其中命名的组件。

如果组件尝试访问密钥环服务时没有可用的密钥环组件或插件，该服务将无法被该组件使用。因此，该组件可能无法初始化或以有限功能初始化。例如，如果`InnoDB`在初始化时发现有加密表空间，它会尝试访问密钥环。如果密钥环不可用，`InnoDB`只能访问未加密的表空间。
