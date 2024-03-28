> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-repair-table.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-repair-table.html)

#### 19.5.1.25 复制和修复表

当在一个损坏或其他受损的表上使用`REPAIR TABLE`语句时，可能会删除无法恢复的行。然而，此语句执行的任何表数据修改都不会被复制，这可能导致源和副本失去同步。因此，在源上的表损坏并使用`REPAIR TABLE`修复之前，应该先停止复制（如果仍在运行），然后比较源和副本的表副本，并准备好在重新启动复制之前手动纠正任何差异。
