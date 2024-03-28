# 17.21.2 故障恢复失败的故障排除

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting-recovery.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting-recovery.html)

从 MySQL 8.0.26 开始，在重做日志恢复完成并且数据字典动态元数据（`srv_dict_metadata`）转移到数据字典表（`dict_table_t`）对象之前，不允许进行检查点和推进检查点 LSN。如果在恢复期间或恢复后（但在数据字典动态元数据转移到数据字典表对象之前）重做日志空间不足，可能需要进行`innodb_force_recovery`重启，至少从`SRV_FORCE_NO_IBUF_MERGE`设置开始，或者在失败的情况下，从`SRV_FORCE_NO_LOG_REDO`设置开始。如果在这种情况下`innodb_force_recovery`重启失败，可能需要从备份中恢复。（Bug #32200595）
