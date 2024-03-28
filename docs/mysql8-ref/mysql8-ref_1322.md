> 原文：[`dev.mysql.com/doc/refman/8.0/en/federated-create-connection.html`](https://dev.mysql.com/doc/refman/8.0/en/federated-create-connection.html)

#### 18.8.2.1 使用 CONNECTION 创建一个 FEDERATED 表

要使用第一种方法，你必须在 `CREATE TABLE` 语句中的引擎类型之后指定 `CONNECTION` 字符串。例如：

```sql
CREATE TABLE federated_table (
    id     INT(20) NOT NULL AUTO_INCREMENT,
    name   VARCHAR(32) NOT NULL DEFAULT '',
    other  INT(20) NOT NULL DEFAULT '0',
    PRIMARY KEY  (id),
    INDEX name (name),
    INDEX other_key (other)
)
ENGINE=FEDERATED
DEFAULT CHARSET=utf8mb4
CONNECTION='mysql://fed_user@remote_host:9306/federated/test_table';
```

注意

`CONNECTION` 取代了一些早期版本的 MySQL 中使用的 `COMMENT`。

`CONNECTION` 字符串包含连接到包含数据物理存储的远程服务器的信息。连接字符串指定服务器名称、登录凭据、端口号和数据库/表信息。在这个例子中，远程表位于服务器 `remote_host` 上，使用端口 9306。名称和端口号应该与你想要用作远程表的远程 MySQL 服务器实例的主机名（或 IP 地址）和端口号匹配。

连接字符串的格式如下：

```sql
*scheme*://*user_name*[:*password*]@*host_name*[:*port_num*]/*db_name*/*tbl_name*
```

其中：

+   *`scheme`*: 一个被识别的连接协议。目前只支持 `mysql` 作为 *`scheme`* 的值。

+   *`user_name`*: 连接的用户名。这个用户必须在远程服务器上创建，并且必须具有执行所需操作（`SELECT`, `INSERT`, `UPDATE`等）所需的适当权限。

+   *`password`*: （可选）*`user_name`* 对应的密码。

+   *`host_name`*: 远程服务器的主机名或 IP 地址。

+   *`port_num`*: （可选）远程服务器的端口号。默认值为 3306。

+   *`db_name`*: 持有远程表的数据库名称。

+   *`tbl_name`*: 远程表的名称。本地表和远程表的名称不必匹配。

示例连接字符串：

```sql
CONNECTION='mysql://username:password@hostname:port/database/tablename'
CONNECTION='mysql://username@hostname/database/tablename'
CONNECTION='mysql://username:password@hostname/database/tablename'
```
