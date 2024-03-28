> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-reserved-words.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-reserved-words.html)

#### 19.5.1.26 复制和保留字

当您尝试从旧源复制到新副本并且在源上使用在新 MySQL 版本（在副本上运行）中被视为保留字的标识符时，可能会遇到问题。例如，在 MySQL 5.7 源上命名为 `rank` 的表列在复制到 MySQL 8.0 副本时可能会导致问题，因为 `RANK` 是 MySQL 8.0 中的保留字。

在这种情况下，即使从复制中排除使用保留字命名的数据库或表，或者具有使用保留字命名的列的表，复制也可能失败，并显示错误 1064“您的 SQL 语法有误...”。这是因为每个 SQL 事件在执行之前必须由副本解析，以便副本知道哪些数据库对象将受到影响。仅在事件解析后，副本才能应用由`--replicate-do-db`、`--replicate-do-table`、`--replicate-ignore-db`和`--replicate-ignore-table`定义的任何过滤规则。

要解决源数据库、表或列名称在副本中被视为保留字的问题，请执行以下操作之一：

+   在源数据库上使用一个或多个`ALTER TABLE`语句来更改任何数据库对象的名称，其中这些名称在副本中被视为保留字，并将使用旧名称的任何 SQL 语句更改为使用新名称。

+   在使用这些数据库对象名称的任何 SQL 语句中，将名称写为带有反引号字符（```）的引用标识符。

有关 MySQL 版本的保留字列表，请参阅*MySQL Server Version Reference*中的 MySQL 8.0 中的关键字和保留字，有关标识符引用规则，请参阅第 11.2 节，“模式对象名称”。
