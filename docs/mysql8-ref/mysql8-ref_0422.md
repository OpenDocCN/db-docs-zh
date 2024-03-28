> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-encrypted-file-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-encrypted-file-plugin.html)

#### 8.4.4.7 使用 keyring_encrypted_file 加密文件型密钥环插件

注意

`keyring_encrypted_file` 插件是包含在 MySQL 企业版中的一个扩展，这是一个商业产品。要了解更多关于商业产品的信息，请参见 [`www.mysql.com/products/`](https://www.mysql.com/products/)。

`keyring_encrypted_file` 密钥环插件将密钥环数据存储在一个加密的、受密码保护的文件中，该文件位于服务器主机本地。

截至 MySQL 8.0.34 版本，该插件已被弃用，并可能在未来的 MySQL 版本中被移除。相反，考虑使用 `component_encrypted_keyring_file` 组件来存储密钥环数据（参见 Section 8.4.4.5, “Using the component_keyring_encrypted_file Encrypted File-Based Keyring Component”）。

警告

对于加密密钥管理，`keyring_encrypted_file` 插件并不是一个旨在符合监管合规性的解决方案。安全标准如 PCI、FIPS 等要求使用密钥管理系统来保护、管理和保护密钥库或硬件安全模块（HSM）中的加密密钥。

要安装 `keyring_encrypted_file`，请使用在 Section 8.4.4.3, “Keyring Plugin Installation” 中找到的一般说明，以及在此处找到的特定于 `keyring_encrypted_file` 的配置信息。

为了在服务器启动过程中可用，`keyring_encrypted_file` 必须使用 `--early-plugin-load` 选项加载。要为加密密钥环数据文件指定密码，请设置 `keyring_encrypted_file_password` 系统变量。（密码是必需的；如果在服务器启动时未指定，`keyring_encrypted_file` 初始化将失败。）`keyring_encrypted_file_data` 系统变量可选地配置 `keyring_encrypted_file` 插件用于数据存储的文件位置。默认值是特定于平台的。要显式配置文件位置，请在启动时设置变量值。例如，在服务器 `my.cnf` 文件中使用这些行，根据需要调整 `.so` 后缀和文件位置以适应您的平台，并替换您选择的密码：

```sql
[mysqld]
early-plugin-load=keyring_encrypted_file.so
keyring_encrypted_file_data=/usr/local/mysql/mysql-keyring/keyring-encrypted
keyring_encrypted_file_password=*password*
```

因为 `my.cnf` 文件在写入时会存储密码，所以应该具有限制模式，并且只能被用来运行 MySQL 服务器的账户访问。

Keyring 操作是事务性的：`keyring_encrypted_file`插件在写操作期间使用备份文件，以确保如果操作失败，可以回滚到原始文件。备份文件的名称与`keyring_encrypted_file_data`系统变量的值相同，后缀为`.backup`。

有关用于配置`keyring_encrypted_file`插件的系统变量的附加信息，请参阅 Section 8.4.4.19, “Keyring 系统变量”。

为确保仅在存在正确的 keyring 存储文件时才刷新密钥，`keyring_encrypted_file`在文件中存储了密钥环的 SHA-256 校验和。在更新文件之前，插件会验证文件是否包含预期的校验和。此外，`keyring_encrypted_file`在写入文件之前使用 AES 加密文件内容，并在读取文件后解密文件内容。

`keyring_encrypted_file`插件支持组成标准 MySQL Keyring 服务接口的函数。这些函数执行的 Keyring 操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用 Section 8.4.4.15, “通用 Keyring 密钥管理函数”中描述的函数。

+   C 接口：在 C 语言代码中，调用 Section 7.6.9.2, “Keyring 服务”中描述的 keyring 服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关`keyring_encrypted_file`允许的关键值特征的信息，请参阅 Section 8.4.4.13, “支持的 Keyring 密钥类型和长度”。
