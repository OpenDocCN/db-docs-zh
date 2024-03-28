> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-sys-config-insert-set-user.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-sys-config-insert-set-user.html)

#### 30.4.2.2 `sys_config_insert_set_user` 触发器

对于通过 `INSERT` 语句添加到 `sys_config` 表中的行，`sys_config_insert_set_user` 触发器将 `set_by` 列设置为当前用户。
