> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks-recover.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks-recover.html)

#### 19.3.3.3 从失败的复制特权检查中恢复

如果针对`PRIVILEGE_CHECKS_USER`账户的特权检查失败，则不会执行事务，并且通道上的复制将停止。错误的详细信息和最后应用的事务记录在性能模式`replication_applier_status_by_worker`表中。按照以下步骤从错误中恢复：

1.  确定导致错误的复制事件，并验证事件是否预期且来自可信任的来源。您可以使用**mysqlbinlog**检索和显示围绕错误发生时间记录的事件。有关如何执行此操作的说明，请参见第 9.5 节，“时间点（增量）恢复” Recovery")。

1.  如果复制的事件是意外的或不是来自已知和可信任的来源，请调查原因。如果您能够确定事件发生的原因，并且没有安全考虑因素，请按照以下描述修复错误。

1.  如果`PRIVILEGE_CHECKS_USER`账户应该被允许执行事务，但已经配置错误，请向账户授予缺失的特权，使用`FLUSH PRIVILEGES`语句或执行**mysqladmin flush-privileges**或**mysqladmin reload**命令重新加载授权表，然后重新启动通道上的复制。

1.  如果需要执行事务并且已经验证了其可信性，但`PRIVILEGE_CHECKS_USER`账户通常不应具有此特权，您可以临时授予`PRIVILEGE_CHECKS_USER`账户所需的特权。在应用了复制事件之后，从账户中删除特权，并采取任何必要的步骤确保事件不会再次发生（如果可以避免）。

1.  如果事务是应该仅在源端而不是在副本上发生的管理操作，或者应该仅在单个复制组成员上发生，可以在停止复制的服务器上跳过事务，然后发出`START REPLICA`来重新启动通道上的复制。为了避免将来出现这种情况，您可以在这些管理语句之前使用`SET sql_log_bin = 0`，之后再使用`SET sql_log_bin = 1`，这样它们就不会在源端记录。

1.  如果事务是一个不应该在源端或副本端发生的 DDL 或 DML 语句，请在停止复制的服务器上跳过该事务，然后在原始发生事务的服务器上手动撤销该事务，最后发出`START REPLICA`以重新启动复制。

要跳过一个事务，如果正在使用 GTIDs，请提交一个具有失败事务 GTID 的空事务，例如：

```sql
SET GTID_NEXT='aaa-bbb-ccc-ddd:N';
BEGIN;
COMMIT;
SET GTID_NEXT='AUTOMATIC';
```

如果没有使用 GTIDs，请发出`SET GLOBAL sql_replica_skip_counter`或`SET GLOBAL sql_slave_skip_counter`语句以跳过该事件。有关使用此替代方法和有关跳过事务的更多详细信息，请参见 Section 19.1.7.3, “Skipping Transactions”。
