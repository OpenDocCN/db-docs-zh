> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-aws-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-aws-plugin.html)

#### 8.4.4.9 使用 keyring_aws 亚马逊网络服务密钥环插件

注意

`keyring_aws`插件是 MySQL 企业版中包含的扩展，这是一个商业产品。要了解更多关于商业产品的信息，请参阅[`www.mysql.com/products/`](https://www.mysql.com/products/)。

`keyring_aws`密钥环插件与亚马逊网络服务密钥管理服务（AWS KMS）通信，作为密钥生成的后端，并使用本地文件进行密钥存储。所有密钥材料均由 AWS 服务器独家生成，而不是由`keyring_aws`生成。

MySQL 企业版可以在 Red Hat Enterprise Linux、SUSE Linux Enterprise Server、Debian、Ubuntu、macOS 和 Windows 上与`keyring_aws`一起使用。MySQL 企业版不支持在以下平台上使用`keyring_aws`：

+   EL6

+   通用 Linux（glibc2.12）

+   SLES 12（MySQL Server 5.7 之后的版本）

+   Solaris

这里的讨论假定您对 AWS 有一般了解，特别是对 KMS 有了解。一些相关信息来源：

+   [AWS 网站](https://aws.amazon.com/kms/)

+   [KMS 文档](https://docs.aws.amazon.com/kms/)

以下各节提供了`keyring_aws`密钥环插件的配置和使用信息：

+   keyring_aws 配置

+   keyring_aws 操作

+   keyring_aws 凭据更改

##### keyring_aws 配置

要安装`keyring_aws`，请使用第 8.4.4.3 节中找到的一般说明，“密钥环插件安装”，以及此处找到的插件特定配置信息。

插件库文件包含`keyring_aws`插件和两个可加载函数，`keyring_aws_rotate_cmk()`和`keyring_aws_rotate_keys()`。

要配置`keyring_aws`，您必须获取一个提供与 AWS KMS 通信凭据的秘密访问密钥，并将其写入配置文件：

1.  创建一个 AWS KMS 账户。

1.  使用 AWS KMS 创建一个秘密访问密钥 ID 和秘密访问密钥。访问密钥用于验证您的身份及您的应用程序的身份。

1.  使用 AWS KMS 账户创建一个 KMS 密钥 ID。在 MySQL 启动时，将`keyring_aws_cmk_id`系统变量设置为 CMK ID 值。此变量是强制性的，没有默认值。（如果需要，可以使用`SET GLOBAL`在运行时更改其值。）

1.  如有必要，创建应放置配置文件的目录。该目录应具有限制模式，并且只能由用于运行 MySQL 服务器的帐户访问。例如，在 Unix 和类 Unix 系统上，要将`/usr/local/mysql/mysql-keyring/keyring_aws_conf`用作文件名，以下命令（以`root`身份执行）创建其父目录并设置目录模式和所有权：

    ```sql
    $> cd /usr/local/mysql
    $> mkdir mysql-keyring
    $> chmod 750 mysql-keyring
    $> chown mysql mysql-keyring
    $> chgrp mysql mysql-keyring
    ```

    在 MySQL 启动时，将`keyring_aws_conf_file`系统变量设置为`/usr/local/mysql/mysql-keyring/keyring_aws_conf`，以指示服务器配置文件的位置。

1.  准备`keyring_aws`配置文件，其中应包含两行：

    +   第 1 行：秘密访问密钥 ID

    +   第 2 行：秘密访问密钥

    例如，如果密钥 ID 为`wwwwwwwwwwwwwEXAMPLE`，密钥为`xxxxxxxxxxxxx/yyyyyyy/zzzzzzzzEXAMPLEKEY`，则配置文件如下所示：

    ```sql
    wwwwwwwwwwwwwEXAMPLE
    xxxxxxxxxxxxx/yyyyyyy/zzzzzzzzEXAMPLEKEY
    ```

在服务器启动过程中可用，必须使用`--early-plugin-load`选项加载`keyring_aws`。`keyring_aws_cmk_id`系统变量是必需的，并配置从 AWS KMS 服务器获取的 KMS 密钥 ID。`keyring_aws_conf_file`和`keyring_aws_data_file`系统变量可选地配置`keyring_aws`插件用于配置信息和数据存储的文件位置。文件位置变量的默认值是特定于平台的。要显式配置位置，请在启动时设置变量值。例如，在服务器`my.cnf`文件中使用这些行，根据需要调整`.so`后缀和文件位置：

```sql
[mysqld]
early-plugin-load=keyring_aws.so
keyring_aws_cmk_id='arn:aws:kms:us-west-2:111122223333:key/abcd1234-ef56-ab12-cd34-ef56abcd1234'
keyring_aws_conf_file=/usr/local/mysql/mysql-keyring/keyring_aws_conf
keyring_aws_data_file=/usr/local/mysql/mysql-keyring/keyring_aws_data
```

要成功启动`keyring_aws`插件，配置文件必须存在并包含有效的秘密访问密钥信息，如前所述初始化。存储文件不需要存在。如果不存在，`keyring_aws`会尝试创建它（以及必要时的父目录）。

有关用于配置`keyring_aws`插件的系统变量的更多信息，请参见第 8.4.4.19 节，“Keyring 系统变量”。

启动 MySQL 服务器并安装与`keyring_aws`插件关联的函数。这是一次性操作，通过执行以下语句执行，根据需要调整`.so`后缀以适应您的平台：

```sql
CREATE FUNCTION keyring_aws_rotate_cmk RETURNS INTEGER
  SONAME 'keyring_aws.so';
CREATE FUNCTION keyring_aws_rotate_keys RETURNS INTEGER
  SONAME 'keyring_aws.so';
```

有关`keyring_aws`函数的更多信息，请参见第 8.4.4.16 节，“插件特定的 Keyring 密钥管理函数”。

##### keyring_aws 操作

在插件启动时，`keyring_aws` 插件从其配置文件中读取 AWS 秘密访问密钥 ID 和密钥。它还将存储文件中包含的任何加密密钥读入其内存缓存中。

在运行过程中，`keyring_aws` 在内存缓存中维护加密密钥，并使用存储文件作为本地持久存储。每个 `keyring_aws` 操作都是事务性的：`keyring_aws` 要么成功地更改内存中的密钥缓存和密钥环存储文件，要么操作失败，密钥环状态保持不变。

为了确保仅在存在正确的密钥环存储文件时才刷新密钥，`keyring_aws` 在文件中存储密钥环的 SHA-256 校验和。在更新文件之前，插件会验证文件是否包含预期的校验和。

`keyring_aws` 插件支持组成标准 MySQL 密钥环服务接口的函数。通过这些函数执行的密钥环操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用 第 8.4.4.15 节，“通用密钥环密钥管理函数” 中描述的函数。

+   C 接口：在 C 语言代码中，调用 第 7.6.9.2 节，“密钥环服务” 中描述的密钥环服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

此外，`keyring_aws_rotate_cmk()` 和 `keyring_aws_rotate_keys()` 函数“扩展”了密钥环插件接口，提供了标准密钥环服务接口未涵盖的与 AWS 相关的功能。只能通过使用 SQL 调用这些函数来访问这些功能。没有相应的 C 语言密钥服务函数。

有关 `keyring_aws` 允许的密钥值特性的信息，请参阅 第 8.4.4.13 节，“支持的密钥环密钥类型和长度”。

##### keyring_aws 凭据更改

假设 `keyring_aws` 插件在服务器启动时已经正确初始化，可以更改用于与 AWS KMS 通信的凭据：

1.  使用 AWS KMS 创建新的秘密访问密钥 ID 和秘密访问密钥。

1.  将新凭据存储在配置文件中（由 `keyring_aws_conf_file` 系统变量命名的文件）。文件格式如前所述。

1.  重新初始化 `keyring_aws` 插件，以便重新读取配置文件。假设新凭据有效，插件应该成功初始化。

    有两种重新初始化插件的方法：

    +   重新启动服务器。这更简单，没有副作用，但不适用于需要最小化服务器停机时间并尽可能少重启的安装。

    +   通过执行以下语句重新初始化插件，根据需要调整`.so`后缀以适应您的平台，而无需重新启动服务器：

        ```sql
        UNINSTALL PLUGIN keyring_aws;
        INSTALL PLUGIN keyring_aws SONAME 'keyring_aws.so';
        ```

        注意

        除了在运行时加载插件外，`INSTALL PLUGIN` 还具有在 `mysql.plugin` 系统表中注册插件的副作用。因此，如果您决定停止使用`keyring_aws`，仅仅从用于启动服务器的选项集中删除`--early-plugin-load`选项是不够的。这样可以阻止插件早期加载，但服务器仍会在启动序列中加载在`mysql.plugin`中注册的插件。

        因此，如果您执行上述描述的`UNINSTALL PLUGIN`加`INSTALL PLUGIN`序列来更改 AWS KMS 凭据，那么要停止使用`keyring_aws`，需要再次执行`UNINSTALL PLUGIN`来取消注册插件，同时删除`--early-plugin-load`选项。
