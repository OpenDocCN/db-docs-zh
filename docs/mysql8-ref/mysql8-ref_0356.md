# 8.1.4 与安全相关的 mysqld 选项和变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/security-options.html`](https://dev.mysql.com/doc/refman/8.0/en/security-options.html)

以下表格显示了影响安全性的**mysqld**选项和系统变量。有关每个选项的描述，请参见第 7.1.7 节，“服务器命令选项”和第 7.1.8 节，“服务器系统变量”。

**表 8.1 安全选项和变量摘要**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| allow-suspicious-udfs | 是 | 是 |  |  |  |  |
| automatic_sp_privileges | 是 | 是 | 是 |  | 全局 | 是 |
| chroot | 是 | 是 |  |  |  |  |
| local_infile | 是 | 是 | 是 |  | 全局 | 是 |
| safe-user-create | 是 | 是 |  |  |  |  |
| secure_file_priv | 是 | 是 | 是 |  | 全局 | 否 |
| skip-grant-tables | 是 | 是 |  |  |  |  |
| skip_name_resolve | 是 | 是 | 是 |  | 全局 | 否 |
| skip_networking | 是 | 是 | 是 |  | 全局 | 否 |
| skip_show_database | 是 | 是 | 是 |  | 全局 | 否 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
