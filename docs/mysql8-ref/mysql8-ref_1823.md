> 译文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-security-mysql-security-procedures.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-security-mysql-security-procedures.html)

#### 25.6.20.3 NDB Cluster 和 MySQL 安全程序

在本节中，我们将讨论 MySQL 标准安全程序，因为它们适用于运行 NDB Cluster。

一般来说，运行 MySQL 安全的任何标准程序也适用于作为 NDB Cluster 的一部分运行 MySQL 服务器。首先，您应始终将 MySQL 服务器作为 `mysql` 操作系统用户运行；这与在标准（非集群）环境中运行 MySQL 没有区别。`mysql` 系统帐户应该是唯一且明确定义的。幸运的是，这是新 MySQL 安装的默认行为。您可以使用系统命令（例如下面显示的命令）验证 **mysqld** 进程是否以 `mysql` 操作系统用户身份运行：

```sql
$> ps aux | grep mysql
root     10467  0.0  0.1   3616  1380 pts/3    S    11:53   0:00 \
  /bin/sh ./mysqld_safe --ndbcluster --ndb-connectstring=localhost:1186
mysql    10512  0.2  2.5  58528 26636 pts/3    Sl   11:53   0:00 \
  /usr/local/mysql/libexec/mysqld --basedir=/usr/local/mysql \
  --datadir=/usr/local/mysql/var --user=mysql --ndbcluster \
  --ndb-connectstring=localhost:1186 --pid-file=/usr/local/mysql/var/mothra.pid \
  --log-error=/usr/local/mysql/var/mothra.err
jon      10579  0.0  0.0   2736   688 pts/0    S+   11:54   0:00 grep mysql
```

如果 **mysqld** 进程以除 `mysql` 之外的任何其他用户身份运行，您应立即关闭它，并以 `mysql` 用户重新启动它。如果此用户在系统上不存在，则应创建 `mysql` 用户帐户，并使此用户成为 `mysql` 用户组的一部分；在这种情况下，您还应确保此系统上的 MySQL 数据目录（使用 `--datadir` 选项设置为 **mysqld**）由 `mysql` 用户拥有，并且 SQL 节点的 `my.cnf` 文件在 `[mysqld]` 部分包含 `user=mysql`。或者，您可以在命令行上使用 `--user=mysql` 启动 MySQL 服务器进程，但最好使用 `my.cnf` 选项，因为您可能会忘记使用命令行选项，从而无意中以其他用户身份运行 **mysqld**。**mysqld_safe** 启动脚本会强制 MySQL 以 `mysql` 用户身份运行。

重要提示

永远不要以系统根用户身份运行 **mysqld**。这样做意味着 MySQL 可能读取系统上的任何文件，因此——如果 MySQL 受到攻击——攻击者可能读取任何文件。

如前一节所述（请参阅 Section 25.6.20.2, “NDB Cluster and MySQL Privileges”），您应该在 MySQL 服务器运行后立即设置 root 密码。您还应删除默认安装的匿名用户帐户。您可以使用以下语句��行这些任务：

```sql
$> mysql -u root

mysql> UPDATE mysql.user
 ->     SET Password=PASSWORD('*secure_password*')
 ->     WHERE User='root';

mysql> DELETE FROM mysql.user
 ->     WHERE User='';

mysql> FLUSH PRIVILEGES;
```

在执行`DELETE`语句时要非常小心，不要忽略`WHERE`子句，否则会风险删除*所有*MySQL 用户。一旦修改了`mysql.user`表，请务必立即运行`FLUSH PRIVILEGES`语句，以便更改立即生效。没有`FLUSH PRIVILEGES`，更改将在下次服务器重新启动时才生效。

注意

许多 NDB 集群实用程序，如**ndb_show_tables**、**ndb_desc**和**ndb_select_all**也可以在没有身份验证的情况下工作，并且可以显示表名、模式和数据。默认情况下，这些程序在 Unix 风格系统上以权限`wxr-xr-x`（755）安装，这意味着任何可以访问`mysql/bin`目录的用户都可以执行它们。

有关这些实用程序的更多信息，请参见第 25.5 节，“NDB 集群程序”。
