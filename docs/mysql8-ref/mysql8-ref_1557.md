> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-online-upgrade-considerations.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-online-upgrade-considerations.html)

#### 20.8.3.1 在线升级注意事项

在升级在线组时，您应考虑以下几点：

+   无论您如何升级您的组，重要的是在他们准备重新加入组之前禁用对组成员的任何写入。

+   当成员停止时，`super_read_only` 变量会自动设置为打开，但此更改不会持久保存。

+   当 MySQL 5.7.22 或 MySQL 8.0.11 尝试加入运行 MySQL 5.7.21 或更低版本的组时，它无法加入该组，因为 MySQL 5.7.21 不会发送其 `lower_case_table_names` 的值。
