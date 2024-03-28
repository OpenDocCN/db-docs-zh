# 27.5 使用视图

> 原文：[`dev.mysql.com/doc/refman/8.0/en/views.html`](https://dev.mysql.com/doc/refman/8.0/en/views.html)

27.5.1 视图语法

27.5.2 视图处理算法

27.5.3 可更新和可插入视图

27.5.4 带有 CHECK OPTION 子句的视图

27.5.5 视图元数据

MySQL 支持视图，包括可更新的视图。视图是存储的查询，当调用时产生一个结果集。视图充当虚拟表。

以下讨论描述了创建和删除视图的语法，并展示了如何使用它们的一些示例。

### 其他资源

+   在处理视图时，您可能会发现[MySQL 用户论坛](https://forums.mysql.com/list.php?20)很有帮助。

+   关于 MySQL 中视图的一些常见问题的答案，请参见第 A.6 节，“MySQL 8.0 FAQ：视图”。

+   使用视图时有一些限制；参见第 27.9 节，“视图限制”。
