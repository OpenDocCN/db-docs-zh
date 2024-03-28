> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-hashicorp-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-hashicorp-plugin.html)

#### 8.4.4.10 使用 HashiCorp Vault 密钥环插件

注意

`keyring_hashicorp` 插件是包含在 MySQL Enterprise Edition 中的扩展，这是一款商业产品。要了解更多关于商业产品的信息，请参阅 [`www.mysql.com/products/`](https://www.mysql.com/products/)。

`keyring_hashicorp` 密钥环插件与 HashiCorp Vault 进行后端存储通信。该插件支持 HashiCorp Vault AppRole 认证。没有密钥信息永久存储在 MySQL 服务器本地存储中（可使用可选的内存中密钥缓存作为中间存储）。随机密钥生成在 MySQL 服务器端执行，然后将密钥存储到 Hashicorp Vault。

`keyring_hashicorp` 插件支持组成标准 MySQL 密钥环服务接口的函数。这些函数执行的密钥环操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用 第 8.4.4.15 节，“通用密钥环密钥管理函数” 中描述的函数。

+   C 接口：在 C 语言代码中，调用 第 7.6.9.2 节，“密钥环服务” 中描述的密钥环服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关 `keyring_hashicorp` 允许的密钥值特性的信息，请参阅 第 8.4.4.13 节，“支持的密钥环密钥类型和长度”。

要安装 `keyring_hashicorp`，请使用 第 8.4.4.3 节，“密钥环插件安装” 中找到的一般说明，以及此处找到的特定于 `keyring_hashicorp` 的配置信息。插件特定配置包括准备连接到 HashiCorp Vault 所需的证书和密钥文件，以及配置 HashiCorp Vault 本身。以下部分提供了必要的说明。

+   证书和密钥准备

+   HashiCorp Vault 设置

+   keyring_hashicorp 配置

##### 证书和密钥准备

`keyring_hashicorp` 插件需要与 HashiCorp Vault 服务器建立安全连接，使用 HTTPS 协议。典型设置包括一组证书和密钥文件：

+   `company.crt`: 组织拥有的自定义 CA 证书。此文件同时被 HashiCorp Vault 服务器和 `keyring_hashicorp` 插件使用。

+   `vault.key`：HashiCorp Vault 服务器实例的私钥。此文件由 HashiCorp Vault 服务器使用。

+   `vault.crt`：HashiCorp Vault 服务器实例的证书。此文件必须由组织 CA 证书签名。

以下说明描述了如何使用 OpenSSL 创建证书和密钥文件。（如果您已经有这些文件，请继续进行 HashiCorp Vault 设置。）所示的说明适用于 Linux 平台，可能需要调整以适用于其他平台。

重要提示

根据这些说明生成的证书是自签名的，可能不太安全。在使用这些文件获得经验后，考虑从注册的证书颁发机构获取证书/密钥材料。

1.  准备公司和 HashiCorp Vault 服务器密钥。

    使用以下命令生成密钥文件：

    ```sql
    openssl genrsa -aes256 -out company.key 4096
    openssl genrsa -aes256 -out vault.key 2048
    ```

    这些命令生成包含公司私钥（`company.key`）和 Vault 服务器私钥（`vault.key`）的文件。这些密钥分别是随机生成的 4,096 位和 2,048 位的 RSA 密钥。

    每个命令都会提示输入密码。为了测试目的，密码不是必需的。要禁用密码，请省略`-aes256`参数。

    密钥文件包含敏感信息，应存储在安全位置。密码（也是敏感信息）稍后会被要求，因此请记下并将其存储在安全位置。

    （可选）使用以下命令检查密钥文件内容和有效性：

    ```sql
    openssl rsa -in company.key -check
    openssl rsa -in vault.key -check
    ```

1.  创建公司 CA 证书。

    使用以下命令创建一个有效期为 365 天的名为`company.crt`的公司 CA 证书文件（在一行上输入命令）：

    ```sql
    openssl req -x509 -new -nodes -key company.key
      -sha256 -days 365 -out company.crt
    ```

    如果在生成密钥时使用了`-aes256`参数进行密钥加密，则在创建 CA 证书时会提示输入公司密钥密码。还会提示输入有关证书持有者（即您或您的公司）的信息，如下所示：

    ```sql
    Country Name (2 letter code) [AU]:
    State or Province Name (full name) [Some-State]:
    Locality Name (eg, city) []:
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:
    Email Address []:
    ```

    用适当的值回答提示。

1.  创建证书签名请求。

    要为新创建的服务器密钥准备一个 HashiCorp Vault 服务器证书，必须创建一个名为`request.conf`的配置文件，其中包含以下行。如果 HashiCorp Vault 服务器不在本地主机上运行，请替换适当的 CN 和 IP 值，并进行任何其他必要的更改。

    ```sql
    [req]
    distinguished_name = vault
    x509_entensions = v3_req
    prompt = no

    [vault]
    C = US
    ST = CA
    L = RWC
    O = Company
    CN = 127.0.0.1

    [v3_req]
    subjectAltName = @alternatives
    authorityKeyIdentifier = keyid,issuer
    basicConstraints = CA:TRUE

    [alternatives]
    IP = 127.0.0.1
    ```

    使用以下命令创建签名请求：

    ```sql
    openssl req -new -key vault.key -config request.conf -out request.csr
    ```

    输出文件（`request.csr`）是一个中间文件，用作创建服务器证书的输入。

1.  创建 HashiCorp Vault 服务器证书。

    使用公司证书（`company.crt`）对 HashiCorp Vault 服务器密钥（`vault.key`）和 CSR（`request.csr`）中的组合信息进行签名，以创建 HashiCorp Vault 服务器证书（`vault.crt`）。使用以下命令执行此操作（在一行上输入命令）：

    ```sql
    openssl x509 -req -in request.csr
      -CA company.crt -CAkey company.key -CAcreateserial
      -out vault.crt -days 365 -sha256
    ```

    要使 `vault.crt` 服务器证书有效，将 `company.crt` 公司证书的内容附加到其中。这是必需的，以便公司证书在请求中与服务器证书一起传递。

    ```sql
    cat company.crt >> vault.crt
    ```

    如果显示 `vault.crt` 文件的内容，它应该如下所示：

    ```sql
    -----BEGIN CERTIFICATE-----
    ... *content of HashiCorp Vault server certificate* ...
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    ... *content of company certificate* ...
    -----END CERTIFICATE-----
    ```

##### HashiCorp Vault 设置

以下说明描述了如何创建一个 HashiCorp Vault 设置，以便测试 `keyring_hashicorp` 插件。

重要

测试设置类似于生产设置，但是 HashiCorp Vault 的生产使用涉及额外的安全考虑，例如使用非自签名证书和将公司证书存储在系统信任存储中。您必须实施任何额外的安全步骤以满足您的运营需求。

这些说明假定在 证书和密钥准备 中创建了证书和密钥文件。如果您没有这些文件，请参阅该部分。

1.  获取 HashiCorp Vault 二进制文件。

    从 [`www.vaultproject.io/downloads.html`](https://www.vaultproject.io/downloads.html) 下载适合您平台的 HashiCorp Vault 二进制文件。

    解压缩存档内容以生成可执行的 **vault** 命令，该命令用于执行 HashiCorp Vault 操作。如有必要，将安装该命令的目录添加到系统路径中。

    （可选）HashiCorp Vault 支持自动补全选项，使其更易于使用。有关更多信息，请参阅 [`learn.hashicorp.com/vault/getting-started/install#command-completion`](https://learn.hashicorp.com/vault/getting-started/install#command-completion)。

1.  创建 HashiCorp Vault 服务器配置文件。

    准备一个名为 `config.hcl` 的配置文件，内容如下。对于 `tls_cert_file`、`tls_key_file` 和 `path` 值，请替换适合您系统的路径名。

    ```sql
    listener "tcp" {
      address="127.0.0.1:8200"
      tls_cert_file="/home/username/certificates/vault.crt"
      tls_key_file="/home/username/certificates/vault.key"
    }

    storage "file" {
      path = "/home/username/vaultstorage/storage"
    }

    ui = true
    ```

1.  启动 HashiCorp Vault 服务器。

    要启动 Vault 服务器，请使用以下命令，其中 `-config` 选项指定刚创建的配置文件的路径：

    ```sql
    vault server -config=config.hcl
    ```

    在此步骤中，您可能会被要求为存储在 `vault.key` 文件中的 Vault 服务器私钥输入密码。

    服务器应该启动，并在控制台上显示一些信息（IP、端口等）。

    为了能够输入剩余的命令，请将 **vault server** 命令放在后台运行或在继续之前打开另一个终端。

1.  初始化 HashiCorp Vault 服务器。

    注意

    在启动 Vault 第一次时，执行本步骤中描述的操作以获取解封密钥和根令牌。后续 Vault 实例重新启动仅需要使用解封密钥进行解封。

    发出以下命令（假设 Bourne shell 语法）：

    ```sql
    export VAULT_SKIP_VERIFY=1
    vault operator init -n 1 -t 1
    ```

    第一个命令使**vault**命令暂时忽略尚未添加到系统信任存储库的公司证书的事实。它弥补了我们的自签名 CA 未添加到该存储库的事实。（对于生产使用，应添加这样的证书。）

    第二个命令创建一个具有单个解封密钥的要求，要求一个解封密钥存在以解封。 （对于生产使用，一个实例将具有多个解封密钥，最多需要输入相同数量的密钥来解封。解封密钥应交付给公司内的密钥保管员。使用单个密钥可能被视为安全问题，因为这允许单个密钥保管员解封保险库。）

    Vault 应该回复有关解封密钥和 root 令牌的信息，以及一些附加文本（实际的解封密钥和 root 令牌值与此处显示的值不同）：

    ```sql
    ...
    Unseal Key 1: I2xwcFQc892O0Nt2pBiRNlnkHzTUrWS+JybL39BjcOE=
    Initial Root Token: s.vTvXeo3tPEYehfcd9WH7oUKz
    ...
    ```

    将解封密钥和 root 令牌存储在安全位置。

1.  解封 HashiCorp Vault 服务器。

    使用此命令解封 Vault 服务器：

    ```sql
    vault operator unseal
    ```

    在提示输入解封密钥时，使用在 Vault 初始化期间先前获得的密钥。

    Vault 应该生成指示设置完成并解封保险库的输出。

1.  登录到 HashiCorp Vault 服务器并验证其状态。

    准备登录为 root 所需的环境变量：

    ```sql
    vault login s.vTvXeo3tPEYehfcd9WH7oUKz
    ```

    在该命令中的令牌值中，用之前在 Vault 初始化期间获得的 root 令牌的内容替换。

    验证 Vault 服务器状态：

    ```sql
    vault status
    ```

    输出应包含以下行（以及其他内容）：

    ```sql
    ...
    Initialized     true
    Sealed          false
    ...
    ```

1.  设置 HashiCorp Vault 身份验证和存储。

    注意

    在运行 Vault 实例时，描述的操作仅在第一次运行时需要。之后不需要重复。

    启用 AppRole 身份验证方法并验证其是否在身份验证方法列表中：

    ```sql
    vault auth enable approle
    vault auth list
    ```

    启用 Vault KeyValue 存储引擎：

    ```sql
    vault secrets enable -version=1 kv
    ```

    为与`keyring_hashicorp`插件一起使用设置一个角色（在一行上输入命令）：

    ```sql
    vault write auth/approle/role/mysql token_num_uses=0
      token_ttl=20m token_max_ttl=30m secret_id_num_uses=0
    ```

1.  添加一个 AppRole 安全策略。

    注意

    在运行 Vault 实例时，描述的操作仅在第一次运行时需要。之后不需要重复。

    准备一个策略，以允许先前创建的角色访问适当的机密。创建一个名为`mysql.hcl`的新文件，内容如下：

    ```sql
    path "kv/mysql/*" {
      capabilities = ["create", "read", "update", "delete", "list"]
    }
    ```

    注意

    在这个示例中，`kv/mysql/`可能需要根据您的本地安装政策和安全要求进行调整。如果需要，无论这些说明中的`kv/mysql/`出现在哪里，都要进行相同的调整。

    将策略文件导入 Vault 服务器以创建名为`mysql-policy`的策略，然后将策略分配给新角色：

    ```sql
    vault policy write mysql-policy mysql.hcl
    vault write auth/approle/role/mysql policies=mysql-policy
    ```

    获取新创建角色的 ID 并将其存储在安全位置：

    ```sql
    vault read auth/approle/role/mysql/role-id
    ```

    为角色生成一个秘密 ID，并将其存储在安全位置：

    ```sql
    vault write -f auth/approle/role/mysql/secret-id
    ```

    生成这些 AppRole 角色 ID 和密钥 ID 凭证后，它们预计会无限期保持有效。不需要再次生成它们，`keyring_hashicorp`插件可以配置它们以供持续使用。有关 AuthRole 身份验证的更多信息，请访问[`www.vaultproject.io/docs/auth/approle.html`](https://www.vaultproject.io/docs/auth/approle.html)。

##### keyring_hashicorp 配置

插件库文件包含`keyring_hashicorp`插件和一个可加载的函数，`keyring_hashicorp_update_config()`。当插件初始化和终止时，它会自动加载和卸载该函数。无需手动加载和卸载该函数。

`keyring_hashicorp`插件支持以下表中显示的配置参数。要指定这些参数，请为相应的系统变量赋值。

| 配置参数 | 系统变量 | 必须 |
| --- | --- | --- |
| HashiCorp 服务器 URL | `keyring_hashicorp_server_url` | 否 |
| AppRole 角色 ID | `keyring_hashicorp_role_id` | 是 |
| AppRole 密钥 ID | `keyring_hashicorp_secret_id` | 是 |
| 存储路径 | `keyring_hashicorp_store_path` | 是 |
| 授权路径 | `keyring_hashicorp_auth_path` | 否 |
| CA 证书文件路径 | `keyring_hashicorp_ca_path` | 否 |
| 缓存控制 | `keyring_hashicorp_caching` | 否 |

为了在服务器启动过程中可用，必须使用`--early-plugin-load`选项加载`keyring_hashicorp`。如前表所示，还必须设置几个与插件相关的系统变量。例如，在服务器的`my.cnf`文件中使用以下行，根据需要调整`.so`后缀和文件位置：

```sql
[mysqld]
early-plugin-load=keyring_hashicorp.so
keyring_hashicorp_role_id='ee3b495c-d0c9-11e9-8881-8444c71c32aa'
keyring_hashicorp_secret_id='0512af29-d0ca-11e9-95ee-0010e00dd718'
keyring_hashicorp_store_path='/v1/kv/mysql'
keyring_hashicorp_auth_path='/v1/auth/approle/login'
```

注意

根据[HashiCorp 文档](https://www.vaultproject.io/api-docs)，所有 API 路由都以协议版本为前缀（在前面的示例中可以看到为`/v1/`，在`keyring_hashicorp_store_path`和`keyring_hashicorp_auth_path`的值中）。如果 HashiCorp 开发新的协议版本，可能需要在配置中将`/v1/`更改为其他内容。

MySQL 服务器使用 AppRole 身份验证对 HashiCorp Vault 进行身份验证。成功的身份验证需要向 Vault 提供两个秘密，一个角色 ID 和一个秘密 ID，类似于用户名和密码的概念。要指定这两个 ID，将它们的值分配给`keyring_hashicorp_role_id`和`keyring_hashicorp_secret_id`系统变量。设置过程还会导致存储路径为`/v1/kv/mysql`，这个值应分配给`keyring_hashicorp_commit_store_path`。

在插件初始化时，`keyring_hashicorp`尝试使用配置数值连接到 HashiCorp Vault 服务器。如果连接成功，插件会将这些数值存储在相应的系统变量中，这些变量的名称中包含`_commit_`。例如，连接成功后，插件会将`keyring_hashicorp_role_id`和`keyring_hashicorp_store_path`的数值存储在`keyring_hashicorp_commit_role_id`和`keyring_hashicorp_commit_store_path`中。

可以通过`keyring_hashicorp_update_config()`函数在运行时重新配置：

1.  使用`SET`语句将所需的新值分配给前表中显示的配置系统变量。这些分配本身对正在进行的插件操作没有影响。

1.  调用`keyring_hashicorp_update_config()`函数会导致插件重新配置并使用新的变量值重新连接到 HashiCorp Vault 服务器。

1.  如果连接成功，插件会将更新后的配置数值存储在相应的系统变量中，这些变量的名称中包含`_commit_`。

例如，如果已将 HashiCorp Vault 重新配置为监听端口 8201 而不是默认的 8200，则像这样重新配置`keyring_hashicorp`：

```sql
mysql> SET GLOBAL keyring_hashicorp_server_url = 'https://127.0.0.1:8201';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT keyring_hashicorp_update_config();
+--------------------------------------+
| keyring_hashicorp_update_config()    |
+--------------------------------------+
| Configuration update was successful. |
+--------------------------------------+
1 row in set (0.03 sec)
```

如果插件在初始化或重新配置期间无法连接到 HashiCorp Vault，并且没有现有连接，则对于字符串值变量，`_commit_`系统变量将设置为`'Not committed'`，对于布尔值变量，将设置为`OFF`。如果插件无法连接但存在现有连接，则该连接保持活动状态，并且`_commit_`变量反映用于它的值。

注意

如果在服务器启动时未设置必需的系统变量，或者发生其他插件初始化错误，初始化将失败。在这种情况下，您可以使用运行时重新配置过程来初始化插件，而无需重新启动服务器。

有关`keyring_hashicorp`插件特定系统变量和功能的其他信息，请参见 Section 8.4.4.19, “Keyring System Variables”和 Section 8.4.4.16, “Plugin-Specific Keyring Key-Management Functions”。
