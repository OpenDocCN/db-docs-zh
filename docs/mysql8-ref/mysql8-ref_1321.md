# 18.8.2 如何创建 FEDERATED 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/federated-create.html`](https://dev.mysql.com/doc/refman/8.0/en/federated-create.html)

18.8.2.1 使用 CONNECTION 创建 FEDERATED 表

18.8.2.2 使用 CREATE SERVER 创建 FEDERATED 表

要创建一个`FEDERATED`表，您应该按照以下步骤进行：

1.  在远程服务器上创建表。或者，记录现有表的表定义，可能使用`SHOW CREATE TABLE`语句。

1.  在本地服务器上创建一个具有相同表定义的表，但添加连接信息，将本地表与远程表链接起来。

例如，您可以在远程服务器上创建以下表：

```sql
CREATE TABLE test_table (
    id     INT(20) NOT NULL AUTO_INCREMENT,
    name   VARCHAR(32) NOT NULL DEFAULT '',
    other  INT(20) NOT NULL DEFAULT '0',
    PRIMARY KEY  (id),
    INDEX name (name),
    INDEX other_key (other)
)
ENGINE=MyISAM
DEFAULT CHARSET=utf8mb4;
```

要创建与远程表联合的本地表，有两种可用选项。您可以创建本地表并指定连接字符串（包含服务器名称、登录、密码），用于使用`CONNECTION`连接到远程表，或者您可以使用之前使用`CREATE SERVER`语句创建的现有连接。

重要

当您创建本地表时，它*必须*具有与远程表相同的字段定义。

注意

通过为主机上的表添加索引，可以提高`FEDERATED`表的性能。优化发生的原因是发送到远程服务器的查询包含`WHERE`子句的内容，并且被发送到远程服务器并在本地执行。这减少了网络流量，否则会请求整个表从服务器传输到本地进行处理。
