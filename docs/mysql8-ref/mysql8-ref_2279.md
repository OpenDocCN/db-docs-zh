# A.6 MySQL 8.0 FAQ：视图

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-views.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-views.html)

A.6.1\. 我在哪里找到涵盖 MySQL 视图的文档？

A.6.2\. 是否有关于 MySQL 视图的讨论论坛？

A.6.3\. 如果基础表被删除或重命名，视图会发生什么？

A.6.4\. MySQL 8.0 是否有表快照？

A.6.5\. MySQL 8.0 是否有物化视图？

A.6.6\. 是否可以向基于连接的视图插入数据？

| **A.6.1.** | 我在哪里找到涵盖 MySQL 视图的文档？ |
| --- | --- |
|  | 查看 Section 27.5，“使用视图”。您可能还会发现[MySQL 用户论坛](https://forums.mysql.com/list.php?20)很有帮助。 |
| **A.6.2.** | 是否有关于 MySQL 视图的讨论论坛？ |
|  | 查看[MySQL 用户论坛](https://forums.mysql.com/list.php?20)。 |
| **A.6.3.** | 如果基础表被删除或重命名，视图会发生什么？ |
|  | 创建视图后，可以删除或更改定义所指的表或视图。要检查此类问题的视图定义，请使用`CHECK TABLE`语句。（参见 Section 15.7.3.2，“CHECK TABLE Statement”.） |
| **A.6.4.** | MySQL 8.0 是否有表快照？ |
|  | 没有。 |
| **A.6.5.** | MySQL 8.0 是否有物化视图？ |
|  | 没有。 |
| **A.6.6.** | 是否可以向基于连接的视图插入数据？ |
|  | 可能，只要您的`INSERT`语句有一个清晰的列列表，表明只涉及一个表。您*不能*通过单个视图插入多个表。 |
