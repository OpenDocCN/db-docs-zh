# 27.3 使用触发器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/triggers.html`](https://dev.mysql.com/doc/refman/8.0/en/triggers.html)

27.3.1 触发器语法和示例

27.3.2 触发器元数据

触发器是与表关联的命名数据库对象，当表发生特定事件时激活。触发器的一些用途包括对要插入表中的值执行检查，或对参与更新的值执行计算。

当语句在关联表中插入、更新或删除行时，触发器被定义为激活。这些行操作是触发器事件。例如，行可以通过 `INSERT` 或 `LOAD DATA` 语句插入，每插入一行触发器就会激活一次。触发器可以设置为在触发事件之前或之后激活。例如，您可以在将每行插入到表中之前或之后激活触发器。

重要提示

MySQL 触发器仅在通过 SQL 语句对表进行更改时激活。这包括对底层可更新视图的基本表进行的更改。触发器不会在通过不向 MySQL 服务器传输 SQL 语句的 API 对表进行的更改时激活。这意味着触发器不会被使用 `NDB` API 进行的更新所激活。

触发器不会因 `INFORMATION_SCHEMA` 或 `performance_schema` 表的更改而激活。这些表实际上是视图，视图上不允许触发器。

以下各节描述了创建和删除触发器的语法，展示了如何使用它们的一些示例，并指示如何获取触发器元数据。

### 其他资源

+   在处理触发器时，您可能会发现 [MySQL 用户论坛](https://forums.mysql.com/list.php?20) 有所帮助。

+   有关 MySQL 中触发器常见问题的答案，请参阅 第 A.5 节 “MySQL 8.0 FAQ: 触发器”。

+   对触发器的使用有一些限制；请参阅 第 27.8 节 “存储程序限制”。

+   触发器的二进制日志记录如 第 27.7 节 “存储程序二进制日志记录” 中所述。
