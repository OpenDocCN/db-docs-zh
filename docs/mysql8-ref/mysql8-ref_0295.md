# 7.6.4 Rewriter 查询重写插件

> 译文：[`dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/rewriter-query-rewrite-plugin.html)

7.6.4.1 安装或卸载 Rewriter 查询重写插件

7.6.4.2 使用 Rewriter 查询重写插件

7.6.4.3 Rewriter 查询重写插件参考

MySQL 支持查询重写插件，可以在服务器执行之前检查并可能修改服务器接收到的 SQL 语句。请参阅 查询重写插件。

MySQL 发行版包括一个名为 `Rewriter` 的后解析查询重写插件以及用于安装插件及其相关元素的脚本。这些元素共同工作，提供语句重写功能：

+   一个名为 `Rewriter` 的服务器端插件检查语句并可能根据其内存中的重写规则缓存对其进行重写。

+   以下语句可能会被重写：

    +   截至 MySQL 8.0.12：支持 `SELECT`、`INSERT`、`REPLACE`、`UPDATE` 和 `DELETE`。

    +   在 MySQL 8.0.12 之前：仅支持 `SELECT`。

    独立语句和预编译语句可能会被重写。出现在视图定义或存储程序中的语句不会被重写。

+   `Rewriter` 插件使用名为 `query_rewrite` 的数据库，其中包含名为 `rewrite_rules` 的表。该表为插件决定是否重写语句提供持久存储的规则。用户通过修改存储在此表中的规则集与插件进行通信。插件通过设置表行的 `message` 列与用户进行通信。

+   `query_rewrite` 数据库包含一个名为 `flush_rewrite_rules()` 的存储过程，将规则表的内容加载到插件中。

+   一个名为 `load_rewrite_rules()` 的可加载函数由 `flush_rewrite_rules()` 存储过程使用。

+   `Rewriter` 插件公开了系统变量，使插件配置和状态变量提供运行时操作信息。在 MySQL 8.0.31 及更高版本中，该插件还支持一个权限（`SKIP_QUERY_REWRITE`），用于保护特定用户的查询免受重写。

以下部分描述了如何安装和使用 `Rewriter` 插件，并提供了其相关元素的参考信息。
