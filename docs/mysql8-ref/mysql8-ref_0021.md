> 原文：[`dev.mysql.com/doc/refman/8.0/en/constraint-invalid-data.html`](https://dev.mysql.com/doc/refman/8.0/en/constraint-invalid-data.html)

#### 1.6.3.3 对无效数据的强制约束

默认情况下，MySQL 8.0 会拒绝无效或不当的数据值，并中止出现这些值的语句。可以通过禁用严格的 SQL 模式（参见第 7.1.11 节，“服务器 SQL 模式”）来改变这种行为，使服务器将其强制转换为有效值以便进行数据输入，但这并不推荐。

旧版本的 MySQL 默认采用宽容的行为；有关此行为的描述，请参阅对无效数据的约束。
