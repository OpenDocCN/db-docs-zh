> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-user-names.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-user-names.html)

#### 19.5.1.38 复制和用户名称长度

MySQL 8.0 中用户名称的最大长度为 32 个字符。 当副本运行的 MySQL 版本早于 5.7 时，长度超过 16 个字符的用户名称的复制将失败，因为这些版本仅支持较短的用户名称。 这仅在从更新的源复制到较旧的副本时发生，这不是推荐的配置。
