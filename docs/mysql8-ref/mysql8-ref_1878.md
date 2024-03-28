# 27.4.3 事件语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/events-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/events-syntax.html)

MySQL 提供了几个用于处理计划事件的 SQL 语句：

+   使用`CREATE EVENT`语句定义新事件。参见 Section 15.1.13, “CREATE EVENT Statement”。

+   通过`ALTER EVENT`语句可以更改现有事件的定义。参见 Section 15.1.3, “ALTER EVENT Statement”。

+   当不再需要或不再需要计划事件时，可以由其定义者使用`DROP EVENT`语句从服务器中删除。参见 Section 15.1.25, “DROP EVENT Statement”。事件是否在其计划结束后继续存在还取决于其`ON COMPLETION`子句，如果有的话。参见 Section 15.1.13, “CREATE EVENT Statement”。

    任何具有数据库上定义事件的`EVENT`权限的用户都可以删除事件。参见 Section 27.4.6, “The Event Scheduler and MySQL Privileges”。
