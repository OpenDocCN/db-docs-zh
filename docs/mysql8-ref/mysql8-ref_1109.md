> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-replicas.html`](https://dev.mysql.com/doc/refman/8.0/en/show-replicas.html)

#### 15.7.7.33 SHOW REPLICAS Statement

```sql
{SHOW REPLICAS}
```

显示当前在源服务器上注册的副本列表。从 MySQL 8.0.22 开始，使用`SHOW REPLICAS`代替从该版本开始弃用的`SHOW SLAVE HOSTS`。在 MySQL 8.0.22 之前的版本中，请使用`SHOW SLAVE HOSTS`。`SHOW REPLICAS`需要`REPLICATION SLAVE`权限。

`SHOW REPLICAS`应在充当复制源的服务器上执行。该语句显示有关作为副本连接的服务器的信息，结果的每一行对应一个副本服务器，如下所示：

```sql
mysql> SHOW REPLICAS;
+------------+-----------+------+-----------+--------------------------------------+
| Server_id  | Host      | Port | Source_id | Replica_UUID                         |
+------------+-----------+------+-----------+--------------------------------------+
|         10 | iconnect2 | 3306 |         3 | 14cb6624-7f93-11e0-b2c0-c80aa9429562 |
|         21 | athena    | 3306 |         3 | 07af4990-f41f-11df-a566-7ac56fdaf645 |
+------------+-----------+------+-----------+--------------------------------------+
```

+   `Server_id`: 副本服务器的唯一服务器 ID，在副本服务器的选项文件中配置，或者使用`--server-id=*`value`*`在命令行上配置。

+   `Host`: 副本服务器的主机名，使用`--report-host`选项在副本上指定。这可能与在操作系统中配置的机器名称不同。

+   `User`: 在副本服务器上指定的副本用户名称，使用`--report-user`选项。只有在源服务器启动时使用`--show-replica-auth-info`或`--show-slave-auth-info`选项时，语句输出才包括此列。

+   `Password`: 副本服务器密码，使用`--report-password`选项在副本上指定。只有在源服务器启动时使用`--show-replica-auth-info`或`--show-slave-auth-info`选项时，语句输出才包括此列。

+   `Port`: 副本服务器正在侦听的源端口，使用`--report-port`选项在副本上指定。

    此列中的零表示未设置副本端口(`--report-port`)。

+   `Source_id`: 副本服务器正在复制的源服务器的唯一服务器 ID。这是在执行`SHOW REPLICAS`的服务器的服务器 ID，因此结果中的每一行都列出相同的值。

+   `Replica_UUID`: 此副本的全局唯一 ID，在副本上生成，并在副本的`auto.cnf`文件中找到。
