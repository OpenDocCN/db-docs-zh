> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-compatibility-upgrade.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-compatibility-upgrade.html)

#### 20.8.1.1 升级过程中的成员版本

在在线升级过程中，如果组处于单主模式，则所有当前未脱机升级的服务器将像以前一样运行。每当需要时，组会选举新的主服务器，遵循第 20.1.3.1 节“单主模式”中描述的选举策略。请注意，如果您要求主服务器在整个过程中保持不变（除非它本身正在升级），则必须首先将所有次要服务器升级到高于或等于目标主服务器版本的版本，然后最后升级主服务器。主服务器不能保持为主服务器，除非它正在运行组中最低的 MySQL 服务器版本。主服务器升级后，您可以使用`group_replication_set_as_primary()`函数重新任命它为主服务器。

如果组处于多主模式，则在升级过程中可用于执行写操作的在线成员较少，因为升级后的成员在升级后以只读模式加入。从 MySQL 8.0.17 开始，这适用于补丁版本之间的升级，对于较低版本，这仅适用于主要版本之间的升级。当所有成员都升级到相同版本时，从 MySQL 8.0.17 开始，它们都会自动切换回读写模式。对于早期版本，您必须在每个应在升级后充当主服务器的成员上手动将`super_read_only`设置为`OFF`。

为了处理问题情况，例如如果您必须回滚升级或在紧急情况下增加组的额外容量，可以允许成员加入在线组，尽管其运行的 MySQL 服务器版本低于其他组成员使用的最低版本。在这种情况下，可以使用 Group Replication 系统变量`group_replication_allow_local_lower_version_join`来覆盖正常的兼容性策略。

重要提示

将`group_replication_allow_local_lower_version_join`设置为`ON`并*不*使新成员与组兼容；这样做允许其加入组，而没有任何防范措施来防止现有成员的不兼容行为。因此，这必须仅在特定情况下小心使用，并且您必须采取额外预防措施，以避免新成员由于正常组活动而失败。有关此变量的更多信息，请参阅其描述。
