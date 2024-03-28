> 原文：[`dev.mysql.com/doc/refman/8.0/en/constraint-primary-key.html`](https://dev.mysql.com/doc/refman/8.0/en/constraint-primary-key.html)

#### 1.6.3.1 主键和唯一索引约束

通常，数据更改语句（如 `INSERT` 或 `UPDATE`）可能违反主键、唯一键或外键约束而导致错误。如果你使用事务性存储引擎如 `InnoDB`，MySQL 会自动回滚该语句。如果你使用非事务性存储引擎，MySQL 会在发生错误的行处停止处理该语句，并留下任何未处理的行。

MySQL 支持 `IGNORE` 关键字用于 `INSERT`、`UPDATE` 等语句。如果使用它，MySQL 会忽略主键或唯一键违规，并继续处理下一行。请参阅你正在使用的语句的部分（15.2.7 “INSERT Statement”，15.2.17 “UPDATE Statement” 等）。

你可以使用 `mysql_info()` C API 函数获取实际插入或更新的行数信息。你也可以使用 `SHOW WARNINGS` 语句。参见 mysql_info()，以及 15.7.7.42 “SHOW WARNINGS Statement”。

`InnoDB` 和 `NDB` 表支持外键。参见 1.6.3.2 “外键约束”。
