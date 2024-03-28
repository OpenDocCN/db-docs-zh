> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-keyring-keys-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-keyring-keys-table.html)

#### 29.12.18.2 密钥环键表

MySQL 服务器支持一个密钥环，使内部服务器组件和插件能够安全地存储敏感信息以供以后检索。参见 第 8.4.4 节，“MySQL 密钥环”。

截至 MySQL 8.0.16，`keyring_keys` 表公开密钥环中密钥的元数据。密钥元数据包括密钥 ID、密钥所有者和后端密钥 ID。`keyring_keys` 表 *不* 公开任何敏感的密钥环数据，如密钥内容。

`keyring_keys` 表包含以下列：

+   `KEY_ID`

    密钥标识符。

+   `KEY_OWNER`

    密钥的所有者。

+   `BACKEND_KEY_ID`

    密钥环后端使用的密钥 ID。

`keyring_keys` 表没有索引。

`截断表` 不允许用于 `keyring_keys` 表。
