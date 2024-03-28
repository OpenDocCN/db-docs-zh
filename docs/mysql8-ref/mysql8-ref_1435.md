> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-create-select.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-create-select.html)

#### 19.5.1.7 复制 CREATE TABLE ... SELECT 语句

当复制`CREATE TABLE ... SELECT`语句时，MySQL 应用以下规则：

+   `CREATE TABLE ... SELECT`总是执行隐式提交（第 15.3.3 节，“导致隐式提交的语句”）。

+   如果目标表不存在，则记录如下。无论是否存在`IF NOT EXISTS`都无关紧要。

    +   `STATEMENT`或`MIXED`格式：该语句按原样记录。

    +   `ROW`格式：该语句被记录为一个`CREATE TABLE`语句，后跟一系列插入行事件。

        在 MySQL 8.0.21 之前，该语句被记录为两个事务。从 MySQL 8.0.21 开始，在支持原子 DDL 的存储引擎上，它被记录为一个事务。更多信息，请参阅第 15.1.1 节，“原子数据定义语句支持”。

+   如果`CREATE TABLE ... SELECT`语句失败，则不会记录任何内容。这包括目标表存在且未使用`IF NOT EXISTS`的情况。

+   如果目标表存在且使用了`IF NOT EXISTS`，MySQL 8.0 会完全忽略该语句；不会插入任何内容或记录日志。

MySQL 8.0 不允许`CREATE TABLE ... SELECT`语句对除了该语句创建的表之外的其他表进行任何更改。
