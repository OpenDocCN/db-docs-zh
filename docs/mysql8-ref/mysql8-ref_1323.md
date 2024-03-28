> 原文：[`dev.mysql.com/doc/refman/8.0/en/federated-create-server.html`](https://dev.mysql.com/doc/refman/8.0/en/federated-create-server.html)

#### 18.8.2.2 使用 CREATE SERVER 创建 FEDERATED 表

如果您在同一服务器上创建多个 `FEDERATED` 表，或者想简化创建 `FEDERATED` 表的过程，可以使用 `CREATE SERVER` 语句来定义服务器连接参数，就像您使用 `CONNECTION` 字符串一样。

`CREATE SERVER` 语句的格式为：

```sql
CREATE SERVER
*server_name*
FOREIGN DATA WRAPPER *wrapper_name*
OPTIONS (*option* [, *option*] ...)
```

*`server_name`* 在创建新的 `FEDERATED` 表时用于连接字符串。

例如，要创建与 `CONNECTION` 字符串相同的服务器连接：

```sql
CONNECTION='mysql://fed_user@remote_host:9306/federated/test_table';
```

您将使用以下语句：

```sql
CREATE SERVER fedlink
FOREIGN DATA WRAPPER mysql
OPTIONS (USER 'fed_user', HOST 'remote_host', PORT 9306, DATABASE 'federated');
```

要创建一个使用此连接的 `FEDERATED` 表，仍然使用 `CONNECTION` 关键字，但指定您在 `CREATE SERVER` 语句中使用的名称。

```sql
CREATE TABLE test_table (
    id     INT(20) NOT NULL AUTO_INCREMENT,
    name   VARCHAR(32) NOT NULL DEFAULT '',
    other  INT(20) NOT NULL DEFAULT '0',
    PRIMARY KEY  (id),
    INDEX name (name),
    INDEX other_key (other)
)
ENGINE=FEDERATED
DEFAULT CHARSET=utf8mb4
CONNECTION='fedlink/test_table';
```

此示例中的连接名称包含连接名称（`fedlink`）和要链接到的表名称（`test_table`），用斜杠分隔。 如果只指定连接名称而没有表名称，则使用本地表的表名称。

有关 `CREATE SERVER` 的更多信息，请参见 Section 15.1.18, “CREATE SERVER Statement”。

`CREATE SERVER` 语句接受与 `CONNECTION` 字符串相同的参数。 `CREATE SERVER` 语句会更新 `mysql.servers` 表中的行。 有关连接字符串中参数、`CREATE SERVER` 语句选项以及 `mysql.servers` 表中列之间的对应关系，请参考以下表格。 供参考，`CONNECTION` 字符串的格式如下：

```sql
*scheme*://*user_name*[:*password*]@*host_name*[:*port_num*]/*db_name*/*tbl_name*
```

| 描述 | `CONNECTION` 字符串 | `CREATE SERVER` 选项 | `mysql.servers` 列 |
| --- | --- | --- | --- |
| 连接方案 | *`scheme`* | `wrapper_name` | `Wrapper` |
| 远程用户 | *`user_name`* | `USER` | `Username` |
| 远程密码 | *`password`* | `PASSWORD` | `Password` |
| 远程主机 | *`host_name`* | `HOST` | `Host` |
| 远程端口 | *`port_num`* | `PORT` | `Port` |
| 远程数据库 | *`db_name`* | `DATABASE` | `Db` |
