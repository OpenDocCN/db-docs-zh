> 原文：[`dev.mysql.com/doc/refman/8.0/en/start-group-replication.html`](https://dev.mysql.com/doc/refman/8.0/en/start-group-replication.html)

#### 15.4.3.1 START GROUP_REPLICATION Statement

```sql
 START GROUP_REPLICATION
          [USER='*user_name*']
          [, PASSWORD='*user_pass*']
          [, DEFAULT_AUTH='*plugin_name*']
```

启动组复制。此语句需要`GROUP_REPLICATION_ADMIN`权限（或已弃用的`SUPER`权限）。如果设置了`super_read_only=ON`并且成员应作为主服务器加入，则一旦 Group Replication 成功启动，`super_read_only`将设置为`OFF`。

参与单主模式组的服务器应使用`skip_replica_start=ON`。否则，服务器不允许作为辅助服务器加入组。

在 MySQL 8.0.21 及更高版本中，您可以使用`USER`、`PASSWORD`和`DEFAULT_AUTH`选项在`START GROUP_REPLICATION`语句中指定分布式恢复的用户凭据，如下所示：

+   `USER`: 用于分布式恢复的复制用户。有关设置此帐户的说明，请参见 Section 20.2.1.3, “User Credentials For Distributed Recovery”。如果指定了`PASSWORD`，则不能指定空字符串或 null 字符串，也不能省略`USER`选项。

+   `PASSWORD`: 复制用户帐户的密码。密码不能加密，但在查询日志中被掩码。

+   `DEFAULT_AUTH`: 用于复制用户帐户的身份验证插件的名称。如果不指定此选项，则假定使用 MySQL 本机身份验证（`mysql_native_password`插件）。此选项作为服务器的提示，并且在分布式恢复的捐赠者上，如果与用户帐户关联的不同插件，则会覆盖它。在 MySQL 8 中创建用户帐户时默认使用的身份验证插件是缓存 SHA-2 身份验证插件（`caching_sha2_password`）。有关身份验证插件的更多信息，请参见 Section 8.2.17, “Pluggable Authentication”。

这些凭据用于`group_replication_recovery`通道上的分布式恢复。当您在`START GROUP_REPLICATION`上指定用户凭据时，这些凭据仅保存在内存中，并且通过`STOP GROUP_REPLICATION`语句或服务器关闭而删除。您必须发出`START GROUP_REPLICATION`语句以再次提供凭据。因此，此方法与根据`group_replication_start_on_boot`系统变量在服务器启动时自动启动 Group Replication 不兼容。

在`START GROUP_REPLICATION`中指定的用户凭据优先于使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）为`group_replication_recovery`通道设置的任何用户凭据。请注意，使用这些语句设置的用户凭据存储在复制元数据存储库中，并且在未指定用户凭据的情况下指定`START GROUP_REPLICATION`时使用，包括如果`group_replication_start_on_boot`系统变量设置为`ON`时的自动启动。为了获得在`START GROUP_REPLICATION`上指定用户凭据的安全性好处，请确保`group_replication_start_on_boot`设置为`OFF`（默认为`ON`），并按照第 20.6.3 节，“保护分布式恢复连接”中的说明清除先前为`group_replication_recovery`通道设置的任何用户凭据。

当成员重新加入复制组时，在组完成兼容性检查并接受其为成员之前，其状态可能显示为`OFFLINE`或`ERROR`。当成员正在赶上组的事务时，其状态为`RECOVERING`。
