# 15.7.6 设置语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-statement.html`](https://dev.mysql.com/doc/refman/8.0/en/set-statement.html)

15.7.6.1 变量赋值的 SET 语法

15.7.6.2 设置 CHARACTER SET 语句

15.7.6.3 设置 NAMES 语句

`SET` 语句有几种形式。那些与特定服务器功能不相关的形式的描述出现在本节的子部分中：

+   `SET *`var_name`* = *`value`*` 允许您为影响服务器或客户端操作的变量分配值。参见 第 15.7.6.1 节，“变量赋值的 SET 语法”。

+   `SET CHARACTER SET` 和 `SET NAMES` 为与服务器当前连接相关的字符集和校对变量分配值。参见 第 15.7.6.2 节，“设置字符集语句”，以及 第 15.7.6.3 节，“设置 NAMES 语句”。

其他形式的描述出现在其他地方，与帮助实现它们的其他语句分组在一起：

+   `SET DEFAULT ROLE` 和 `SET ROLE` 设置用户账户的默认角色和当前角色。参见 第 15.7.1.9 节，“设置默认角色语句”，以及 第 15.7.1.11 节，“设置角色语句”。

+   `SET PASSWORD` 用于分配账户密码。参见 第 15.7.1.10 节，“设置密码语句”。

+   `SET RESOURCE GROUP` 为线程分配资源组。参见 第 15.7.2.4 节，“设置资源组语句”。

+   `SET TRANSACTION ISOLATION LEVEL` 用于设置事务处理的隔离级别。参见 第 15.3.7 节，“设置事务语句”。
