# 15.1.18 CREATE SERVER 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-server.html`](https://dev.mysql.com/doc/refman/8.0/en/create-server.html)

```sql
CREATE SERVER *server_name*
    FOREIGN DATA WRAPPER *wrapper_name*
    OPTIONS (*option* [, *option*] ...)

*option*: {
    HOST *character-literal*
  | DATABASE *character-literal*
  | USER *character-literal*
  | PASSWORD *character-literal*
  | SOCKET *character-literal*
  | OWNER *character-literal*
  | PORT *numeric-literal*
}
```

这个语句创建了一个用于`FEDERATED`存储引擎的服务器定义。`CREATE SERVER`语句在`mysql`数据库的`servers`表中创建了一行新记录。此语句需要`SUPER`权限。

`*`server_name`*`应该是对服务器的唯一引用。服务器定义在服务器的范围内是全局的，不可能将服务器定义限定到特定数据库。`*`server_name`*`最大长度为 64 个字符（超过 64 个字符的名称会被静默截断），且不区分大小写。您可以将名称指定为带引号的字符串。

`*`wrapper_name`*`是一个标识符，可以用单引号引起来。

对于每个`*`option`*`，您必须指定字符文字或数字文字。字符文字为 UTF-8 编码，支持最大长度为 64 个字符，默认为空字符串。字符串文字会被静默截断为 64 个字符。数字文字必须是 0 到 9999 之间的数字，默认值为 0。

注意

`OWNER`选项目前未应用，对创建的服务器连接的所有权或操作没有影响。

`CREATE SERVER`语句在`mysql.servers`表中创建一个条目，以后可以在创建`FEDERATED`表时与`CREATE TABLE`语句一起使用。您指定的选项用于填充`mysql.servers`表中的列。表列包括`Server_name`、`Host`、`Db`、`Username`、`Password`、`Port`和`Socket`。

例如：

```sql
CREATE SERVER s
FOREIGN DATA WRAPPER mysql
OPTIONS (USER 'Remote', HOST '198.51.100.106', DATABASE 'test');
```

一定要指定建立与服务器连接所需的所有选项。用户名、主机名和数据库名是必需的。可能还需要其他选项，比如密码。

存储在表中的数据可在创建到`FEDERATED`表的连接时使用：

```sql
CREATE TABLE t (s1 INT) ENGINE=FEDERATED CONNECTION='s';
```

欲了解更多信息，请参阅第 18.8 节，“FEDERATED 存储引擎”。

`CREATE SERVER`会导致隐式提交。请参阅第 15.3.3 节，“导致隐式提交的语句”。

无论使用的日志格式如何，`CREATE SERVER`都不会写入二进制日志。
