> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-oci-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-oci-plugin.html)

#### 8.4.4.12 使用 Oracle Cloud 基础设施 Vault Keyring 插件

注意

`keyring_oci` 插件是包含在 MySQL 企业版中的扩展，这是一个商业产品。要了解更多关于商业产品的信息，请参阅 [`www.mysql.com/products/`](https://www.mysql.com/products/)。

`keyring_oci` 插件是一个与 Oracle Cloud 基础设施 Vault 通信的 keyring 插件，用于后端存储。没有密钥信息永久存储在 MySQL 服务器本地存储中。所有密钥都存储在 Oracle Cloud 基础设施 Vault 中，使得这个插件非常适合 Oracle Cloud 基础设施 MySQL 客户管理他们的 MySQL 企业版密钥。

截至 MySQL 8.0.31，此插件已被弃用，并可能在将来的 MySQL 版本中被移除。相反，考虑使用 `component_keyring_oci` 组件来存储 keyring 数据（请参阅 第 8.4.4.11 节，“使用 Oracle Cloud 基础设施 Vault Keyring 组件”）。

`keyring_oci` 插件支持组成标准 MySQL Keyring 服务接口的函数。这些函数执行的 keyring 操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用 第 8.4.4.15 节，“通用 Keyring 密钥管理函数” 中描述的函数。

+   C 接口：在 C 语言代码中，调用 第 7.6.9.2 节，“Keyring 服务” 中描述的 keyring 服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关 `keyring_oci` 允许的密钥值特性的信息，请参阅 第 8.4.4.13 节，“支持的 Keyring 密钥类型和长度”。

要安装 `keyring_oci`，请使用 第 8.4.4.3 节，“Keyring 插件安装” 中的通用说明，以及此处找到的特定于 `keyring_oci` 的配置信息。插件特定的配置涉及设置多个系统变量以指示 Oracle Cloud 基础设施资源的名称或值。

假设您熟悉 Oracle Cloud 基础设施概念，但在设置资源以供 `keyring_oci` 插件使用时，以下文档可能会有所帮助：

+   [Vault 概述](https://docs.cloud.oracle.com/iaas/Content/KeyManagement/Concepts/keyoverview.htm)

+   [资源标识符](https://docs.cloud.oracle.com/en-us/iaas/Content/General/Concepts/identifiers.htm)

+   [必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)

+   [管理密钥](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingkeys.htm)

+   [管理区段](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm)

+   [管理保险柜](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)

+   [管理机密](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingsecrets.htm)

`keyring_oci` 插件支持下表中显示的配置参数。要指定这些参数，请为相应的系统变量赋值。

| 配置参数 | 系统变量 | 强制 |
| --- | --- | --- |
| 用户 OCID | `keyring_oci_user` | 是 |
| 租户 OCID | `keyring_oci_tenancy` | 是 |
| 区段 OCID | `keyring_oci_compartment` | 是 |
| 保险柜 OCID | `keyring_oci_virtual_vault` | 是 |
| 主密钥 OCID | `keyring_oci_master_key` | 是 |
| 加密服务器端点 | `keyring_oci_encryption_endpoint` | 是 |
| 密钥管理服务器端点 | `keyring_oci_management_endpoint` | 是 |
| 保险柜服务器端点 | `keyring_oci_vaults_endpoint` | 是 |
| 机密服务器端点 | `keyring_oci_secrets_endpoint` | 是 |
| RSA 私钥文件 | `keyring_oci_key_file` | 是 |
| RSA 私钥指纹 | `keyring_oci_key_fingerprint` | 是 |
| CA 证书捆绑文件 | `keyring_oci_ca_certificate` | 否 |
| 配置参数 | 系统变量 | 强制 |

为了在服务器启动过程中可用，必须使用 `--early-plugin-load` 选项加载 `keyring_oci`。如前表所示，几个与插件相关的系统变量是强制性的，也必须设置：

+   Oracle Cloud 基础设施广泛使用 Oracle Cloud ID（OCID）来指定资源，而且几个 `keyring_oci` 参数指定要使用的资源的 OCID 值。因此，在使用 `keyring_oci` 插件之前，必须满足这些先决条件：

    +   必须存在用于连接到 Oracle Cloud Infrastructure 的用户。如有必要，请创建用户并将用户 OCID 分配给 `keyring_oci_user` 系统变量。

    +   必须存在要使用的 Oracle Cloud Infrastructure 租户，以及租户内的 MySQL 专用区和专用区内的保险库。如有必要，请创建这些资源，并确保用户已启用以使用它们。将租户、专用区和保险库的 OCID 分配给 `keyring_oci_tenancy`、`keyring_oci_compartment` 和 `keyring_oci_virtual_vault` 系统变量。

    +   必须存在用于加密的主密钥。如有必要，请创建它并将其 OCID 分配给 `keyring_oci_master_key` 系统变量。

+   必须指定几个服务器端点。这些端点是特定于保险库的，Oracle Cloud Infrastructure 在创建保险库时分配它们。从保险库详细信息页面获取它们的值，并将其分配给 `keyring_oci_encryption_endpoint`、`keyring_oci_management_endpoint`、`keyring_oci_vaults_endpoint` 和 `keyring_oci_secrets_endpoint` 系统变量。

+   Oracle Cloud Infrastructure API 使用 RSA 公钥/私钥对进行身份验证。要创建此密钥对并获取密钥指纹，请使用 [必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm) 中的说明。将私钥文件名和密钥指纹分配给 `keyring_oci_key_file` 和 `keyring_oci_key_fingerprint` 系统变量。

除了必填的系统变量外，`keyring_oci_ca_certificate` 可以选择设置以指定用于对等身份验证的证书颁发机构（CA）证书包文件。

重要提示

如果从 Oracle Cloud Infrastructure 控制台复制参数，则复制的值可能包含初始的 `https://` 部分。在设置相应的 `keyring_oci` 系统变量时，请省略该部分。

例如，要加载和配置 `keyring_oci`，请在服务器的 `my.cnf` 文件中使用以下行（根据需要调整平台的 `.so` 后缀和文件位置）：

```sql
[mysqld]
early-plugin-load=keyring_oci.so
keyring_oci_user=ocid1.user.oc1..*longAlphaNumericString*
keyring_oci_tenancy=ocid1.tenancy.oc1..*longAlphaNumericString*
keyring_oci_compartment=ocid1.compartment.oc1..*longAlphaNumericString*
keyring_oci_virtual_vault=ocid1.vault.oc1.iad.*shortAlphaNumericString*.*longAlphaNumericString*
keyring_oci_master_key=ocid1.key.oc1.iad.*shortAlphaNumericString*.*longAlphaNumericString*
keyring_oci_encryption_endpoint=*shortAlphaNumericString*-crypto.kms.us-ashburn-1.oraclecloud.com
keyring_oci_management_endpoint=*shortAlphaNumericString*-management.kms.us-ashburn-1.oraclecloud.com
keyring_oci_vaults_endpoint=vaults.us-ashburn-1.oci.oraclecloud.com
keyring_oci_secrets_endpoint=secrets.vaults.us-ashburn-1.oci.oraclecloud.com
keyring_oci_key_file=*file_name*
keyring_oci_key_fingerprint=12:34:56:78:90:ab:cd:ef:12:34:56:78:90:ab:cd:ef
```

有关 `keyring_oci` 插件特定系统变量的更多信息，请参见 第 8.4.4.19 节，“Keyring 系统变量”。

`keyring_oci` 插件不支持运行时重新配置，其系统变量均无法在运行时修改。要更改配置参数，请执行以下操作：

+   修改 `my.cnf` 文件中的参数设置，或者对于持久化到 `mysqld-auto.conf` 的参数，使用 `SET PERSIST_ONLY`。

+   重新启动服务器。
