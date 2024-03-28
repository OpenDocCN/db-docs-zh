> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-functions-plugin-specific.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-functions-plugin-specific.html)

#### 8.4.4.16 插件特定的密钥环密钥管理函数

对于每个密钥环插件特定的函数，本节描述了其目的、调用顺序和返回值。有关通用密钥环函数的信息，请参见第 8.4.4.15 节，“通用密钥环密钥管理函数”。

+   `keyring_aws_rotate_cmk()`

    关联的密钥环插件：`keyring_aws`

    `keyring_aws_rotate_cmk()`旋转 AWS KMS 密钥。旋转仅更改 AWS KMS 用于后续数据密钥加密操作的密钥。AWS KMS 保留先前的 CMK 版本，因此使用先前 CMK 生成的密钥在旋转后仍然可解密。

    旋转会更改 AWS KMS 内部使用的 CMK 值，但不会更改用于引用它的 ID，因此在调用`keyring_aws_rotate_cmk()`之后无需更改`keyring_aws_cmk_id`系统变量。

    此函数需要`SUPER`权限。

    参数：

    无。

    返回值：

    返回 1 表示成功，或者返回`NULL`和错误表示失败。

+   `keyring_aws_rotate_keys()`

    关联的密钥环插件：`keyring_aws`

    `keyring_aws_rotate_keys()`用于旋转存储在由`keyring_aws_data_file`系统变量命名的`keyring_aws`存储文件中的密钥。旋转将文件中的每个密钥发送到 AWS KMS 进行重新加密，使用`keyring_aws_cmk_id`系统变量的值作为 CMK 值，并将新加密的密钥存储在文件中。

    `keyring_aws_rotate_keys()`在以下情况下用于密钥重新加密：

    +   在旋转 CMK 之后；也就是在调用`keyring_aws_rotate_cmk()`函数之后。

    +   在更改`keyring_aws_cmk_id`系统变量为不同密钥值之后。

    此函数需要`SUPER`权限。

    参数：

    无。

    返回值：

    返回 1 表示成功，或者返回`NULL`和错误表示失败。

+   `keyring_hashicorp_update_config()`

    关联的密钥环插件：`keyring_hashicorp`

    调用`keyring_hashicorp_update_config()`函数会导致`keyring_hashicorp`执行运行时重新配置，如 keyring_hashicorp Configuration 中所述。

    此函数需要`SYSTEM_VARIABLES_ADMIN`权限，因为它修改全局系统变量。

    参数：

    无。

    返回值：

    返回成功时字符串`'Configuration update was successful.'`，失败时返回`'Configuration update failed.'`。
