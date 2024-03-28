# 15.7.7 显示语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/show.html`](https://dev.mysql.com/doc/refman/8.0/en/show.html)

15.7.7.1 显示二进制日志语句

15.7.7.2 显示二进制日志事件语句

15.7.7.3 显示字符集语句

15.7.7.4 显示排序规则语句

15.7.7.5 显示列语句

15.7.7.6 显示创建数据库语句

15.7.7.7 显示创建事件语句

15.7.7.8 显示创建函数语句

15.7.7.9 显示创建存储过程语句

15.7.7.10 显示创建表语句

15.7.7.11 显示创建触发器语句

15.7.7.12 显示创建用户语句

15.7.7.13 显示创建视图语句

15.7.7.14 显示数据库语句

15.7.7.15 显示引擎语句

15.7.7.16 显示引擎语句

15.7.7.17 显示错误语句

15.7.7.18 显示事件语句

15.7.7.19 显示函数代码语句

15.7.7.20 显示函数状态语句

15.7.7.21 显示授权语句

15.7.7.22 显示索引语句

15.7.7.23 显示主状态语句

15.7.7.24 显示打开表语句

15.7.7.25 显示插件语句

15.7.7.26 显示权限语句

15.7.7.27 显示存储过程代码语句

15.7.7.28 显示存储过程状态语句

15.7.7.29 显示进程列表语句

15.7.7.30 显示概要语句

15.7.7.31 显示概要语句

15.7.7.32 显示中继日志事件语句

15.7.7.33 显示副本语句

15.7.7.34 显示从属主机 | 显示副本语句

15.7.7.35 显示副本状态语句

15.7.7.36 显示从属 | 副本状态语句

15.7.7.37 显示状态语句

[15.7.7.38 显示表状态语句] (show-table-status.html)

15.7.7.39 显示表语句

15.7.7.40 显示触发器语句

15.7.7.41 显示变量��句

15.7.7.42 显示警告语句

`显示`有许多形式，提供关于数据库、表、列或服务器状态信息的信息。本节描述了以下内容：

```sql
SHOW {BINARY | MASTER} LOGS
SHOW BINLOG EVENTS [IN '*log_name*'] [FROM *pos*] [LIMIT [*offset*,] *row_count*]
SHOW {CHARACTER SET | CHARSET} [*like_or_where*]
SHOW COLLATION [*like_or_where*]
SHOW [FULL] COLUMNS FROM *tbl_name* [FROM *db_name*] [*like_or_where*]
SHOW CREATE DATABASE *db_name*
SHOW CREATE EVENT *event_name*
SHOW CREATE FUNCTION *func_name*
SHOW CREATE PROCEDURE *proc_name*
SHOW CREATE TABLE *tbl_name*
SHOW CREATE TRIGGER *trigger_name*
SHOW CREATE VIEW *view_name*
SHOW DATABASES [*like_or_where*]
SHOW ENGINE *engine_name* {STATUS | MUTEX}
SHOW [STORAGE] ENGINES
SHOW ERRORS [LIMIT [*offset*,] *row_count*]
SHOW EVENTS
SHOW FUNCTION CODE *func_name*
SHOW FUNCTION STATUS [*like_or_where*]
SHOW GRANTS FOR *user*
SHOW INDEX FROM *tbl_name* [FROM *db_name*]
SHOW MASTER STATUS
SHOW OPEN TABLES [FROM *db_name*] [*like_or_where*]
SHOW PLUGINS
SHOW PROCEDURE CODE *proc_name*
SHOW PROCEDURE STATUS [*like_or_where*]
SHOW PRIVILEGES
SHOW [FULL] PROCESSLIST
SHOW PROFILE [*types*] [FOR QUERY *n*] [OFFSET *n*] [LIMIT *n*]
SHOW PROFILES
SHOW RELAYLOG EVENTS [IN '*log_name*'] [FROM *pos*] [LIMIT [*offset*,] *row_count*]
SHOW {REPLICAS | SLAVE HOSTS}
SHOW {REPLICA | SLAVE} STATUS [FOR CHANNEL *channel*]
SHOW [GLOBAL | SESSION] STATUS [*like_or_where*]
SHOW TABLE STATUS [FROM *db_name*] [*like_or_where*]
SHOW [FULL] TABLES [FROM *db_name*] [*like_or_where*]
SHOW TRIGGERS [FROM *db_name*] [*like_or_where*]
SHOW [GLOBAL | SESSION] VARIABLES [*like_or_where*]
SHOW WARNINGS [LIMIT [*offset*,] *row_count*]

*like_or_where*: {
    LIKE '*pattern*'
  | WHERE *expr*
}
```

如果给定的 `SHOW` 语句的语法包括一个 `LIKE '*`pattern`*'` 部分，`'*`pattern`*'` 是一个字符串，可以包含 SQL 中的 `%` 和 `_` 通配符。该模式对于将语句输出限制为匹配值非常有用。

几个 `SHOW` 语句还接受一个 `WHERE` 子句，以提供更灵活的指定要显示哪些行的方式。请参阅 第 28.8 节，“SHOW Statements 的扩展”。

在 `SHOW` 语句的结果中，用户名称和主机名使用反引号（`）引用。

许多 MySQL API（如 PHP）使您可以将从 `SHOW` 语句返回的结果视为从 `SELECT` 返回的结果集一样处理；请参阅 第三十一章，*Connectors and APIs*，或者查看您的 API 文档以获取更多信息。此外，您可以在 SQL 中使用来自 `INFORMATION_SCHEMA` 数据库表查询的结果，而这是您无法轻松使用 `SHOW` 语句的结果所能做到的。请参阅 第二十八章，*INFORMATION_SCHEMA Tables*。
