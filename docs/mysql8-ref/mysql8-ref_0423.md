> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-okv-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-okv-plugin.html)

#### 8.4.4.8 使用 keyring_okv KMIP 插件

注意

`keyring_okv`插件是包含在 MySQL 企业版中的扩展，这是一款商业产品。要了解更多关于商业产品的信息，请参阅[`www.mysql.com/products/`](https://www.mysql.com/products/)。

密钥管理互操作性协议（KMIP）使密钥管理服务器和其客户端之间能够通信。`keyring_okv`密钥环插件使用 KMIP 1.1 协议作为 KMIP 后端的客户端进行安全通信。密钥材料仅由后端生成，而不是由`keyring_okv`生成。该插件与以下支持 KMIP 的产品配合使用：

+   Oracle Key Vault

+   Gemalto SafeNet KeySecure Appliance

+   Townsend Alliance Key Manager

+   Entrust KeyControl

每个 MySQL 服务器实例必须单独注册为 KMIP 的客户端。如果两个或更多 MySQL 服务器实例使用相同的凭据集，它们可能会干扰彼此的功能。

`keyring_okv`插件支持组成标准 MySQL 密钥环服务接口的功能。这些功能执行的密钥环操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用第 8.4.4.15 节“通用密钥环密钥管理功能”中描述的函数。

+   C 接口：在 C 语言代码中，调用第 7.6.9.2 节“密钥环服务”中描述的密钥环服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关`keyring_okv`允许的密钥值特性的信息，请参阅第 8.4.4.13 节“支持的密钥环密钥类型和长度”。

要安装`keyring_okv`，请使用第 8.4.4.3 节“密钥环插件安装”中的通用说明，以及此处找到的特定于`keyring_okv`的配置信息。

+   通用 keyring_okv 配置

+   为 Oracle Key Vault 配置 keyring_okv

+   为 Gemalto SafeNet KeySecure Appliance 配置 keyring_okv

+   为 Townsend Alliance Key Manager 配置 keyring_okv

+   为 Entrust KeyControl 配置 keyring_okv

+   保护 keyring_okv 密钥文件的密码

##### 一般 keyring_okv 配置

无论`keyring_okv`插件使用哪个 KMIP 后端进行密钥存储，`keyring_okv_conf_dir`系统变量都配置了`keyring_okv`用于其支持文件的目录位置。默认值为空，因此必须在插件能够与 KMIP 后端通信之前设置变量以命名一个正确配置的目录。否则，`keyring_okv`在服务器启动期间向错误日志写入一条消息，说明无法通信：

```sql
[Warning] Plugin keyring_okv reported: 'For keyring_okv to be
initialized, please point the keyring_okv_conf_dir variable to a directory
containing Oracle Key Vault configuration file and ssl materials'
```

`keyring_okv_conf_dir`变量必须命名一个包含以下项目的目录：

+   `okvclient.ora`：包含`keyring_okv`与之通信的 KMIP 后端详细信息的文件。

+   `ssl`：包含用于与 KMIP 后端建立安全连接所需的证书和密钥文件的目录：`CA.pem`、`cert.pem`和`key.pem`。如果密钥文件受密码保护，`ssl`目录可以包含一个名为`password.txt`的单行文本文件，其中包含解密密钥文件所需的密码。

`keyring_okv`正常工作需要`okvclient.ora`文件和包含证书和密钥文件的`ssl`目录。用于将这些文件填充到配置目录的过程取决于与`keyring_okv`一起使用的 KMIP 后端，如其他地方所述。

由`keyring_okv`用作其支持文件位置的配置目录应具有严格的模式，并且只能由用于运行 MySQL 服务器的帐户访问。例如，在 Unix 和类 Unix 系统上，要使用`/usr/local/mysql/mysql-keyring-okv`目录，以下命令（以`root`身份执行）创建目录并设置其模式和所有权：

```sql
cd /usr/local/mysql
mkdir mysql-keyring-okv
chmod 750 mysql-keyring-okv
chown mysql mysql-keyring-okv
chgrp mysql mysql-keyring-okv
```

要在服务器启动过程中可用，必须使用`--early-plugin-load`选项加载`keyring_okv`。此外，设置`keyring_okv_conf_dir`系统变量，告诉`keyring_okv`在哪里找到其配置目录。例如，在服务器`my.cnf`文件中使用以下行，根据需要调整平台的`.so`后缀和目录位置：

```sql
[mysqld]
early-plugin-load=keyring_okv.so
keyring_okv_conf_dir=/usr/local/mysql/mysql-keyring-okv
```

有关`keyring_okv_conf_dir`的更多信息，请参见第 8.4.4.19 节，“Keyring 系统变量”。

##### 为 Oracle Key Vault 配置 keyring_okv

这里的讨论假定您熟悉 Oracle Key Vault。一些相关信息来源：

+   [Oracle Key Vault 网站](http://www.oracle.com/technetwork/database/options/key-management/overview/index.html)

+   [Oracle Key Vault 文档](http://www.oracle.com/technetwork/database/options/key-management/documentation/index.html)

在 Oracle Key Vault 术语中，使用 Oracle Key Vault 存储和检索安全对象的客户端称为端点。要与 Oracle Key Vault 通信，必须注册为端点并通过下载和安装端点支持文件进行注册。请注意，必须为每个 MySQL Server 实例注册一个单独的端点。如果两个或更多个 MySQL Server 实例使用相同的端点，则它们可能会干扰彼此的功能。

以下过程简要总结了设置`keyring_okv`以与 Oracle Key Vault 一起使用的过程：

1.  创建`keyring_okv`插件使用的配置目录。

1.  在 Oracle Key Vault 中注册一个端点以获取注册令牌。

1.  使用注册令牌获取`okvclient.jar`客户端软件下载。

1.  安装客户端软件以填充包含 Oracle Key Vault 支持文件的`keyring_okv`配置目录。

使用以下过程配置`keyring_okv`和 Oracle Key Vault 一起工作。此描述仅概述了如何与 Oracle Key Vault 交互。有关详细信息，请访问[Oracle Key Vault](http://www.oracle.com/technetwork/database/options/key-management/overview/index.html)网站并查阅 Oracle Key Vault 管理员指南。

1.  创建包含 Oracle Key Vault 支持文件的配置目录，并确保`keyring_okv_conf_dir`系统变量设置为该目录的名称（有关详细信息，请参阅通用 keyring_okv 配置）。

1.  以具有系统管理员角色的用户身份登录到 Oracle Key Vault 管理控制台。

1.  选择“端点”选项卡以到达端点页面。在端点页面上，点击“添加”。

1.  提供所需的端点信息并点击“注册”。端点类型应为其他。成功注册将产生一个注册令牌。

1.  从 Oracle Key Vault 服务器注销。

1.  再次连接到 Oracle Key Vault 服务器，这次不需要登录。使用端点注册令牌进行注册并请求`okvclient.jar`软件下载。将此文件保存到您的系统中。

1.  使用以下命令安装`okvclient.jar`文件（您必须具有 JDK 1.4 或更高版本）：

    ```sql
    java -jar okvclient.jar -d *dir_name* [-v]
    ```

    `-d`选项后面的目录名称是安装提取文件的位置。如果给出`-v`选项，则会产生日志信息，如果命令失败，这些信息可能会有用。

    当命令要求输入 Oracle Key Vault 端点密码时，请不要提供密码。而是按下**Enter**键。（结果是端点连接到 Oracle Key Vault 时不需要密码。）

    前面的命令会生成一个 `okvclient.ora` 文件，应该位于前面 **java -jar** 命令中 `-d` 选项指定的目录下的这个位置：

    ```sql
    install_dir/conf/okvclient.ora
    ```

    预期的文件内容包括类似于这样的行：

    ```sql
    SERVER=*host_ip*:*port_num*
    STANDBY_SERVER=*host_ip*:*port_num*
    ```

    `SERVER` 变量是必需的，`STANDBY_SERVER` 变量是可选的。`keyring_okv` 插件尝试与由 `SERVER` 变量指定的主机上运行的服务器通信，如果失败则退回到 `STANDBY_SERVER`。

    注意

    如果现有文件不是这种格式，则创建一个新文件，其中包含前面示例中显示的行。此外，在运行 **okvutil** 命令之前，请考虑备份 `okvclient.ora` 文件。根据需要恢复文件。

    从 MySQL 8.0.29 开始，您可以指定多个备用服务器（最多 64 个）。如果这样做，`keyring_okv` 插件会迭代它们，直到建立连接，如果无法建立连接则会失败。要添加额外的备用服务器，请编辑 `okvclient.ora` 文件，将服务器的 IP 地址和端口号作为 `STANDBY_SERVER` 变量的值以逗号分隔的列表指定。例如：

    ```sql
    STANDBY_SERVER=*host_ip*:*port_num*,*host_ip*:*port_num*,*host_ip*:*port_num*,*host_ip*:*port_num*
    ```

    确保备用服务器列表保持简短、准确和最新，并删除不再有效的服务器。每次连接尝试都会等待 20 秒，因此长列表中存在无效服务器会显著影响 `keyring_okv` 插件的连接时间，从而影响服务器启动时间。

1.  转到 Oracle Key Vault 安装程序目录，并通过运行此命令测试设置：

    ```sql
    okvutil/bin/okvutil list
    ```

    输出应该类似于这样：

    ```sql
    Unique ID                               Type            Identifier
    255AB8DE-C97F-482C-E053-0100007F28B9	Symmetric Key	-
    264BF6E0-A20E-7C42-E053-0100007FB29C	Symmetric Key	-
    ```

    对于一个全新的 Oracle Key Vault 服务器（一个没有任何密钥的服务器），输出看起来像这样，以表明保险库中没有密钥：

    ```sql
    no objects found
    ```

1.  使用此命令从 `okvclient.jar` 文件中提取包含 SSL 材料的 `ssl` 目录：

    ```sql
    jar xf okvclient.jar ssl
    ```

1.  将 Oracle Key Vault 支持文件（`okvclient.ora` 文件和 `ssl` 目录）复制到配置目录中。

1.  （可选）如果您希望对密钥文件进行密码保护，请使用 Password-Protecting the keyring_okv Key File 中的说明。

完成前述过程后，重新启动 MySQL 服务器。它会加载 `keyring_okv` 插件，并且 `keyring_okv` 使用其配置目录中的文件与 Oracle Key Vault 进行通信。

##### 配置 keyring_okv 以适用于 Gemalto SafeNet KeySecure Appliance

Gemalto SafeNet KeySecure Appliance 使用 KMIP 协议（版本 1.1 或 1.2）。支持 KMIP 1.1 的 `keyring_okv` 密钥环插件可以使用 KeySecure 作为其 KMIP 后端进行密钥环存储。

使用以下步骤配置 `keyring_okv` 和 KeySecure 一起工作。描述仅总结了如何与 KeySecure 交互。有关详细信息，请参阅 [KeySecure 用户指南](https://www2.gemalto.com/aws-marketplace/usage/vks/uploadedFiles/Support_and_Downloads/AWS/007-012362-001-keysecure-appliance-user-guide-v7.1.0.pdf) 中的添加 KMIP 服务器部分。

1.  创建包含 KeySecure 支持文件的配置目录，并确保 `keyring_okv_conf_dir` 系统变量设置为该目录的名称（有关详细信息，请参阅 通用 keyring_okv 配置）。

1.  在配置目录中，创建一个名为 `ssl` 的子目录，用于存储所需的 SSL 证书和密钥文件。

1.  在配置目录中，创建一个名为 `okvclient.ora` 的文件。它应具有以下格式：

    ```sql
    SERVER=*host_ip*:*port_num*
    STANDBY_SERVER=*host_ip*:*port_num*
    ```

    例如，如果 KeySecure 在主机 198.51.100.20 上运行，并在端口 9002 上监听，同时也在备用主机 203.0.113.125 上运行，并在端口 8041 上监听，那么 `okvclient.ora` 文件如下所示：

    ```sql
    SERVER=198.51.100.20:9002
    STANDBY_SERVER=203.0.113.125:8041
    ```

    从 MySQL 8.0.29 开始，您可以指定多个备用服务器（最多 64 个）。如果这样做，`keyring_okv` 插件会对它们进行迭代，直到建立连接，如果无法建立连接则会失败。要添加额外的备用服务器，请编辑 `okvclient.ora` 文件，将服务器的 IP 地址和端口号作为逗号分隔列表指定在 `STANDBY_SERVER` 变量的值中。例如：

    ```sql
    STANDBY_SERVER=*host_ip*:*port_num*,*host_ip*:*port_num*,*host_ip*:*port_num*,*host_ip*:*port_num*
    ```

    确保备用服务器列表保持简短、准确和最新，并删除不再有效的服务器。每次连接尝试都会等待 20 秒，因此长时间的无效服务器列表会显著影响 `keyring_okv` 插件的连接时间，从而影响服务器启动时间。

1.  以具有证书颁发机构访问权限的管理员凭据登录到 KeySecure 管理控制台。

1.  导航至安全性 >> 本地 CA 并创建本地证书颁发机构（CA）。

1.  转到受信任的 CA 列表。选择默认并点击属性。然后选择受信任证书颁发机构列表的编辑并添加刚创建的 CA。

1.  下载 CA 并将其保存在 `ssl` 目录中，文件名为 `CA.pem`。

1.  导航至安全性 >> 证书请求并创建证书。然后您可以下载一个包含证书 PEM 文件的压缩 **tar** 文件。

1.  从下载的文件中提取 PEM 文件。例如，如果文件名为 `csr_w_pk_pkcs8.gz`，请使用以下命令进行解压缩和解包：

    ```sql
    tar zxvf csr_w_pk_pkcs8.gz
    ```

    提取操作会生成两个文件：`certificate_request.pem` 和 `private_key_pkcs8.pem`。

1.  使用此 **openssl** 命令解密私钥并创建名为 `key.pem` 的文件：

    ```sql
    openssl pkcs8 -in private_key_pkcs8.pem -out key.pem
    ```

1.  将 `key.pem` 文件复制到 `ssl` 目录中。

1.  将`certificate_request.pem`中的证书请求复制到剪贴板中。

1.  导航至安全性 >> 本地 CA。选择之前创建的相同 CA（用于创建`CA.pem`文件的那个），然后点击签署请求。将剪贴板中的证书请求粘贴进去，选择客户端的证书目的（密钥环是 KeySecure 的客户端），然后点击签署请求。结果是在新页面中使用所选 CA 签署的证书。

1.  将签署的证书复制到剪贴板，然后将剪贴板内容保存为名为`cert.pem`的文件，保存在`ssl`目录中。

1.  （可选）如果您希望对密钥文件进行密码保护，请使用密码保护 keyring_okv 密钥文件中的说明。

完成上述过程后，重新启动 MySQL 服务器。它会加载`keyring_okv`插件，并且`keyring_okv`会使用其配置目录中的文件与 KeySecure 进行通信。

##### 配置用于 Townsend Alliance Key Manager 的`keyring_okv`

Townsend Alliance Key Manager 使用 KMIP 协议。`keyring_okv`密钥环插件可以使用 Alliance Key Manager 作为其 KMIP 后端进行密钥环存储。有关更多信息，请参见[用于 MySQL 的 Alliance Key Manager](https://www.townsendsecurity.com/product/encryption-key-management-mysql)。

##### 配置用于 Entrust KeyControl 的`keyring_okv`

Entrust KeyControl 使用 KMIP 协议。`keyring_okv`密钥环插件可以使用 Entrust KeyControl 作为其 KMIP 后端进行密钥环存储。有关更多信息，请参见[Oracle MySQL 和 Entrust KeyControl 与 nShield HSM 集成指南](https://www.entrust.com/-/media/documentation/integration-guides/oracle-mysql-enterprise-keycontrol-nshield-ig.pdf)。

##### 保护 keyring_okv 密钥文件的密码

您可以选择使用密码保护密钥文件，并提供包含密码的文件以启用密钥文件的解密。要这样做，请切换到`ssl`目录并执行以下步骤：

1.  加密`key.pem`密钥文件。例如，使用以下命令，并在提示时输入加密密码：

    ```sql
    $> openssl rsa -des3 -in key.pem -out key.pem.new
    Enter PEM pass phrase:
    Verifying - Enter PEM pass phrase:
    ```

1.  将加密密码保存在名为`password.txt`的单行文本文件中，保存在`ssl`目录中。

1.  验证加密密钥文件是否可以使用以下命令解密。解密后的文件应显示在控制台上：

    ```sql
    $> openssl rsa -in key.pem.new -passin file:password.txt
    ```

1.  删除原始的`key.pem`文件，并将`key.pem.new`重命名为`key.pem`。

1.  更改新的`key.pem`文件和`password.txt`文件的所有权和访问模式，以确保它们具有与`ssl`目录中其他文件相同的限制。
