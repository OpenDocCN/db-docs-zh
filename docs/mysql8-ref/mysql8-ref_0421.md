> [`dev.mysql.com/doc/refman/8.0/en/keyring-file-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-file-plugin.html)

#### 8.4.4.6 使用基于文件的 keyring_file 密钥环插件

`keyring_file` 密钥环插件将密钥环数据存储在服务器主机上的一个文件中。

截至 MySQL 8.0.34 版本，此插件已被弃用，并可能在将来的 MySQL 版本中被移除。相反，考虑使用 `component_keyring_file` 组件来存储密钥环数据（参见 第 8.4.4.4 节，“使用 component_keyring_file 基于文件的密钥环组件”）。

警告

对于加密密钥管理，`keyring_file` 插件并不是一个符合监管合规性的解决方案。诸如 PCI、FIPS 等安全标准要求使用密钥管理系统来在密钥保险库或硬件安全模块（HSM）中安全、管理和保护加密密钥。

要安装 `keyring_file`，请使用 第 8.4.4.3 节，“密钥环插件安装” 中找到的一般说明，以及此处找到的特定于 `keyring_file` 的配置信息。

为了在服务器启动过程中可用，必须使用 `--early-plugin-load` 选项加载 `keyring_file`。`keyring_file_data` 系统变量可选地配置 `keyring_file` 插件用于数据存储的文件位置。默认值是特定于平台的。要显式配置文件位置，请在启动时设置变量值。例如，在服务器的 `my.cnf` 文件中使用以下行，根据需要调整 `.so` 后缀和文件位置：

```sql
[mysqld]
early-plugin-load=keyring_file.so
keyring_file_data=/usr/local/mysql/mysql-keyring/keyring
```

密钥环操作是事务性的：`keyring_file` 插件在写操作期间使用备份文件，以确保如果操作失败，可以回滚到原始文件。备份文件的名称与 `keyring_file_data` 系统变量的值相同，后缀为 `.backup`。

有关 `keyring_file_data` 的更多信息，请参阅 第 8.4.4.19 节，“密钥环系统变量”。

为了确保只有在正确的密钥环存储文件存在时才刷新密钥，`keyring_file` 在文件中存储了密钥环的 SHA-256 校验和。在更新文件之前，插件会验证文件是否包含预期的校验和。

`keyring_file` 插件支持组成标准 MySQL 密钥环服务接口的函数。这些函数执行的密钥环操作可在两个级别访问：

+   SQL 接口：在 SQL 语句中，调用第 8.4.4.15 节，“通用密钥环密钥管理函数”中描述的函数。

+   C 接口：在 C 语言代码中，调用第 7.6.9.2 节，“密钥环服务”中描述的密钥环服务函数。

示例（使用 SQL 接口）：

```sql
SELECT keyring_key_generate('MyKey', 'AES', 32);
SELECT keyring_key_remove('MyKey');
```

有关`keyring_file`允许的密钥值特性的信息，请参阅第 8.4.4.13 节，“支持的密钥环密钥类型和长度”。
