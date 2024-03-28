# 15.7.5 CLONE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone.html`](https://dev.mysql.com/doc/refman/8.0/en/clone.html)

```sql
CLONE *clone_action*

*clone_action*: {
    LOCAL DATA DIRECTORY [=] '*clone_dir*';
  | INSTANCE FROM '*user*'@'*host*':*port*
    IDENTIFIED BY '*password*'
    [DATA DIRECTORY [=] '*clone_dir*']
    [REQUIRE [NO] SSL]
}
```

`CLONE` 语句用于在本地或从远程 MySQL 服务器实例克隆数据。要使用`CLONE` 语法，必须安装克隆插件。请参见 Section 7.6.7, “克隆插件”。

`CLONE LOCAL DATA DIRECTORY` 语法从本地 MySQL 数据目录克隆数据到 MySQL 服务器实例运行的同一服务器或节点上的目录。`'clone_dir'` 目录是数据克隆到的本地目录的完整路径。需要绝对路径。指定的目录不能存在，但指定的路径必须是现有路径。MySQL 服务器需要必要的写入权限以创建指定目录。有关更多信息，请参见 Section 7.6.7.2, “本地克隆数据”。

`CLONE INSTANCE` 语法从远程 MySQL 服务器实例（捐赠方）克隆数据并将其传输到启动克隆操作的 MySQL 实例（接收方）。

+   `*`user`*` 是在捐赠 MySQL 服务器实例上的克隆用户。

+   `*`host`*` 是捐赠 MySQL 服务器实例的`hostname`地址。不支持 Internet Protocol version 6 (IPv6) 地址格式。可以使用 IPv6 地址的别名。IPv4 地址可以直接使用。

+   `*`port`*` 是捐赠 MySQL 服务器实例的`port`号。 (不支持由`mysqlx_port`指定的 X 协议端口。也不支持通过 MySQL Router 连接到捐赠 MySQL 服务器实例。)

+   `IDENTIFIED BY '*`password`*'` 指定捐赠 MySQL 服务器实例上克隆用户的密码。

+   `DATA DIRECTORY [=] '*`clone_dir`*'` 是一个可选子句，用于指定在接收方用于克隆数据的目录。如果您不想删除接收方数据目录中的现有数据，请使用此选项。需要绝对路径，并且目录不能存在。MySQL 服务器必须具有必要的写入权限以创建目录。

    当不使用可选的 `DATA DIRECTORY [=] '*`clone_dir`*'` 子句时，克隆操作会删除接收方数据目录中的现有数据，用克隆数据替换它，并在之后自动重新启动服务器。

+   `[REQUIRE [NO] SSL]`明确指定在通过网络传输克隆数据时是否使用加密连接。如果无法满足明确规定，将返回错误。如果未指定 SSL 子句，克隆尝试默认建立加密连接，如果安全连接尝试失败，则回退到不安全连接。无论是否指定此子句，克隆加密数据时都需要安全连接。有关更多信息，请参见为克隆配置加密连接。

关于从远程 MySQL 服务器实例克隆数据的更多信息，请参见 Section 7.6.7.3, “Cloning Remote Data”。
