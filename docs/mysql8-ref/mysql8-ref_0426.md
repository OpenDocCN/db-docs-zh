> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-oci-component.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-oci-component.html)

#### 8.4.4.11 使用 Oracle Cloud Infrastructure Vault 密钥环组件

注意

Oracle Cloud Infrastructure Vault 密钥环组件包含在 MySQL Enterprise Edition 中，这是一个商业产品。要了解更多关于商业产品的信息，请参见 [`www.mysql.com/products/`](https://www.mysql.com/products/)。

`component_keyring_oci` 是与 Oracle Cloud Infrastructure Vault 进行后端存储通信的组件基础设施的一部分。没有任何密钥信息永久存储在 MySQL 服务器本地存储中。所有密钥都存储在 Oracle Cloud Infrastructure Vault 中，使得这个组件非常适合 Oracle Cloud Infrastructure MySQL 客户管理他们的 MySQL Enterprise Edition 密钥。

在 MySQL 8.0.24 中，MySQL Keyring 开始从插件过渡到使用组件基础设施。MySQL 8.0.31 中引入的 `component_keyring_oci` 是这一努力的延续。有关更多信息，请参见 密钥环组件与密钥环插件。

注意

一次只能启用一个密钥环组件或插件。启用多个密钥环组件或插件是不受支持的，结果可能不如预期。

要使用 `component_keyring_oci` 进行密钥库管理，您必须：

1.  编写一个清单，告诉服务器加载 `component_keyring_oci`，如 Section 8.4.4.2, “密钥环组件安装” 中所述。

1.  编写一个 `component_keyring_oci` 的配置文件，如此处所述。

在编写清单和配置文件之后，只要您指定相同的配置选项集来初始化密钥环组件，您就应该能够访问使用 `keyring_oci` 插件创建的密钥。`component_keyring_oci` 的内置向后兼容性简化了从密钥环插件迁移到组件的过程。

+   配置注意事项

+   验证组件安装

+   Vault 密钥环组件使用

##### 配置注意事项

当初始化时，`component_keyring_oci` 会读取全局配置文件，或者全局配置文件与本地配置文件配对：

+   该组件尝试从安装组件库文件的目录（即服务器插件目录）读取其全局配置文件。

+   如果全局配置文件指示使用本地配置文件，则组件将尝试从数据目录读取其本地配置文件。

+   尽管全局和本地配置文件位于不同的目录中，但文件名在两个位置都是 `component_keyring_oci.cnf`。

+   如果不存在配置文件，则会出现错误。没有有效配置时，`component_keyring_oci` 无法初始化。

本地配置文件允许设置多个服务器实例以使用 `component_keyring_oci`，因此每个服务器实例的组件配置特定于给定的数据目录实例。这使得可以为每个实例使用不同的 Oracle Cloud Infrastructure 保险库来使用相同的密钥环组件。

假设您熟悉 Oracle Cloud Infrastructure 概念，但在设置资源以供 `component_keyring_oci` 使用时，以下文档可能会有所帮助：

+   [保险库概述](https://docs.cloud.oracle.com/iaas/Content/KeyManagement/Concepts/keyoverview.htm)

+   [所需密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)

+   [密钥管理](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingkeys.htm)

+   [管理区段](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm)

+   [管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)

+   [管理秘密](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingsecrets.htm)

`component_keyring_oci` 配置文件具有以下属性：

+   配置文件必须采用有效的 JSON 格式。

+   配置文件允许这些配置项：

    +   `"read_local_config"`：此项仅允许在全局配置文件中。如果该项不存在，则组件仅使用全局配置文件。如果该项存在，则其值为 `true` 或 `false`，表示组件是否应从本地配置文件中读取配置信息。

        如果全局配置文件中同时存在 `"read_local_config"` 项和其他项，则组件首先检查 `"read_local_config"` 项的值：

        +   如果值为 `false`，组件将处理全局配置文件中的其他项，并忽略本地配置文件。

        +   如果值为 `true`，组件将忽略全局配置文件中的其他项，并尝试读取本地配置文件。

    +   `“user”`：`component_keyring_oci` 用于连接的 Oracle Cloud Infrastructure 用户的 OCID。在使用 `component_keyring_oci` 之前，用户帐户必须存在并被授予访问权限以使用配置的 Oracle Cloud Infrastructure 租户、区段和保险库资源。要从控制台获取用户 OCID，请使用 [所需密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm) 中的说明。

        此值为必填项。

    +   `“tenancy”`: Oracle Cloud Infrastructure 租户的 OCID，`component_keyring_oci`用作 MySQL 区域的位置。在使用`component_keyring_oci`之前，如果租户不存在，则必须创建一个租户。要从控制台获取租户 OCID，请使用[必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)中的说明。

        此值为必填项。

    +   `“compartment”`: 租户区域的 OCID，`component_keyring_oci`用作 MySQL 密钥的位置。在使用`component_keyring_oci`之前，如果 MySQL 区域或子区域不存在，则必须创建一个。此区域不应包含保险库密钥或保险库机密。它不应被除 MySQL Keyring 之外的系统使用。有关管理区域和获取 OCID 的信息，请参阅[管理区域](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm)。

        此值为必填项。

    +   `“virtual_vault”`: Oracle Cloud Infrastructure Vault 的 OCID，`component_keyring_oci`用于加密操作。在使用`component_keyring_oci`之前，如果 MySQL 区域不存在，则必须在 MySQL 区域中创建一个新的保险库。（或者，您可以重用位于 MySQL 区域的父区域中的现有保险库。）区域用户只能看到并使用其各自区域中的密钥。有关创建保险库和获取保险库 OCID 的信息，请参阅[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)。

        此值为必填项。

    +   `“encryption_endpoint”`: Oracle Cloud Infrastructure 加密服务器的端点，`component_keyring_oci`用于为新密钥生成加密或编码信息（密文）。加密端点是与保险库相关的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取端点 OCID，请查看您的 keyring_oci 保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

        此值为必填项。

    +   `"management_endpoint"`: Oracle Cloud Infrastructure 密钥管理服务器的端点，`component_keyring_oci`用于列出现有密钥的端点。密钥管理端点是与保险库相关的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取端点 OCID，请查看您的 keyring_oci 保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

        此值为必填项。

    +   `“vaults_endpoint”`: Oracle Cloud Infrastructure 保险库服务器的终端点，`component_keyring_oci`用于获取秘密的值。保险库终端点是特定于保险库的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取终端点 OCID，请查看您的 keyring_oci 保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

        此值为必填项。

    +   `“secrets_endpoint”`: Oracle Cloud Infrastructure 秘密服务器的终端点，`component_keyring_oci`用于列出、创建和撤销秘密。秘密终端点是特定于保险库的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取终端点 OCID，请查看您的 keyring_oci 保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

        此值为必填项。

    +   `“master_key”`: Oracle Cloud Infrastructure 主加密密钥的 OCID，`component_keyring_oci`用于加密秘密。在使用`component_keyring_oci`之前，如果不存在，您必须为 Oracle Cloud Infrastructure 区段创建一个加密密钥。为生成的密钥提供一个特定于 MySQL 的名称，并且不要将其用于其他目的。有关密钥创建的信息，请参阅[管理密钥](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingkeys.htm)。

        此值为必填项。

    +   `“key_file”`: 包含 RSA 私钥的文件路径名，`component_keyring_oci`用于 Oracle Cloud Infrastructure 身份验证。您还必须使用控制台上传相应的 RSA 公钥。控制台显示密钥指纹值，您可以使用它来设置`"key_fingerprint"`值。有关生成和上传 API 密钥的信息，请参阅[必需的密钥和 OCIDs](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)。

        此值为必填项。

    +   `“key_fingerprint”`: RSA 私钥的指纹，`component_keyring_oci`用于 Oracle Cloud Infrastructure 身份验证。在创建 API 密钥时获取密钥指纹，请执行此命令：

        ```sql
        openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem | openssl md5 -c
        ```

        或者，从控制台获取指纹，当您上传 RSA 公钥时，控制台会自动显示指纹。有关获取密钥指纹的信息，请参阅[必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)。

        此值为必填项。

    +   `“ca_certificate”`：`component_keyring_oci`组件用于 Oracle Cloud Infrastructure 证书验证的 CA 证书捆绑文件的路径名。该文件包含一个或多个用于对等验证的证书。如果未指定文件，则使用系统上安装的默认 CA 捆绑。如果值设置为`disabled`（区分大小写），`component_keyring_oci`不执行证书验证。

鉴于上述配置文件属性，要配置`component_keyring_oci`，请在安装`component_keyring_oci`库文件的目录中创建一个名为`component_keyring_oci.cnf`的全局配置文件，并可选地在数据目录中创建一个名为`component_keyring_oci.cnf`的本地配置文件。

##### 验证组件安装

执行任何特定于组件的配置后，启动服务器。通过检查性能模式`keyring_component_status`表来验证组件安装：

```sql
mysql> SELECT * FROM performance_schema.keyring_component_status;
+---------------------+--------------------------------------------------------------------+
| STATUS_KEY          | STATUS_VALUE                                                       |
+---------------------+--------------------------------------------------------------------+
| Component_name      | component_keyring_oci                                              |
| Author              | Oracle Corporation                                                 |
| License             | PROPRIETARY                                                        |
| Implementation_name | component_keyring_oci                                              |
| Version             | 1.0                                                                |
| Component_status    | Active                                                             |
| user                | ocid1.user.oc1..aaaaaaaasqly<...>                                  |
| tenancy             | ocid1.tenancy.oc1..aaaaaaaai<...>                                  |
| compartment         | ocid1.compartment.oc1..aaaaaaaah2swh<...>                          |
| virtual_vault       | ocid1.vault.oc1.iad.bbo5xyzkaaeuk.abuwcljtmvxp4r<...>              |
| master_key          | ocid1.key.oc1.iad.bbo5xyzkaaeuk.abuwcljrbsrewgap<...>              |
| encryption_endpoint | bbo5xyzkaaeuk-crypto.kms.us-<...>                                  |
| management_endpoint | bbo5xyzkaaeuk-management.kms.us-<...>                              |
| vaults_endpoint     | vaults.us-<...>                                                    |
| secrets_endpoint    | secrets.vaults.us-<...>                                            |
| key_file            | ~/.oci/oci_api_key.pem                                             |
| key_fingerprint     | ca:7c:e1:fa:86:b6:40:af:39:d6<...>                                 |
| ca_certificate      | disabled                                                           |
+---------------------+--------------------------------------------------------------------+
```

`Component_status`值为`Active`表示组件初始化成功。

如果组件无法加载，服务器启动将失败。检查服务器错误日志以获取诊断消息。如果组件加载但由于配置问题而无法初始化，则服务器启动，但`Component_status`值为`Disabled`。检查服务器错误日志，纠正配置问题，并使用`ALTER INSTANCE RELOAD KEYRING`语句重新加载配置。

可以查询 MySQL 服务器以获取现有密钥列表。要查看存在哪些密钥，请检查性能模式`keyring_keys`表。

```sql
mysql> SELECT * FROM performance_schema.keyring_keys;
+-----------------------------+--------------+----------------+
| KEY_ID                      | KEY_OWNER    | BACKEND_KEY_ID |
+-----------------------------+--------------+----------------+
| audit_log-20210322T130749-1 |              |                |
| MyKey                       | me@localhost |                |
| YourKey                     | me@localhost |                |
+-----------------------------+--------------+----------------+
```

##### 保险库密钥环组件用法

`component_keyring_oci`支持组成标准 MySQL 密钥环服务接口的函数。这些函数执行的密钥环操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用第 8.4.4.15 节，“通用密钥环密钥管理函数”中描述的函数。

+   C 接口：在 C 语言代码中，调用第 7.6.9.2 节，“密钥环服务”中描述的密钥环服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关`component_keyring_oci`允许的密钥值特性的信息，请参见第 8.4.4.13 节，“支持的密钥类型和长度”。
