# 25.6.10 NDB Cluster 的 MySQL 服务器用法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mysqld.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mysqld.html)

**mysqld**是传统的 MySQL 服务器进程。要与 NDB Cluster 一起使用，**mysqld**需要构建支持`NDB`存储引擎，就像在[`dev.mysql.com/downloads/`](https://dev.mysql.com/downloads/)提供的预编译二进制文件中一样。如果您从源代码构建 MySQL，您必须使用`-DWITH_NDB=1`或（已弃用）`-DWITH_NDBCLUSTER=1`选项调用**CMake**以包含对`NDB`的支持。

有关从源代码编译 NDB Cluster 的更多信息，请参阅 Section 25.3.1.4, “Building NDB Cluster from Source on Linux”，以及 Section 25.3.2.2, “Compiling and Installing NDB Cluster from Source on Windows”。

（有关**mysqld**选项和变量的信息，除了本节讨论的内容外，与 NDB Cluster 相关的内容，请参阅 Section 25.4.3.9, “MySQL Server Options and Variables for NDB Cluster”。）

如果**mysqld**二进制文件已经构建支持集群，`NDBCLUSTER`存储引擎仍然默认禁用。您可以使用以下两种选项之一来启用此引擎：

+   在启动**mysqld**时，在命令行上使用`--ndbcluster`作为启动选项。

+   在您的`my.cnf`文件的`[mysqld]`部分中插入一行包含`ndbcluster`。

通过在 MySQL Monitor（**mysql**）中发出 `SHOW ENGINES` 语句，您可以轻松验证服务器是否启用了 `NDBCLUSTER` 存储引擎。您应该在 `NDBCLUSTER` 的行中看到 `Support` 值为 `YES`。如果在此行中看到 `NO` 或者输出中没有显示这样的行，则表示您没有运行启用了 `NDB` 的 MySQL 版本。如果在此行中看到 `DISABLED`，则需要按照刚才描述的两种方式之一启用它。

要读取集群配置数据，MySQL 服务器至少需要三个信息：

+   MySQL 服务器自身的集群节点 ID

+   管理服务器的主机名或 IP 地址

+   它可以连接到管理服务器的 TCP/IP 端口号

节点 ID 可以动态分配，因此不一定需要显式指定。

**mysqld** 参数 `ndb-connectstring` 用于在启动 **mysqld** 时的命令行或在 `my.cnf` 中指定连接字符串。连接字符串包含管理服务器的主机名或 IP 地址，以及其使用的 TCP/IP 端口。

在下面的示例中，`ndb_mgmd.mysql.com` 是管理服务器所在的主机，管理服务器在端口 1186 上监听集群消息：

```sql
$> mysqld --ndbcluster --ndb-connectstring=ndb_mgmd.mysql.com:1186
```

更多关于连接字符串的信息，请参阅 Section 25.4.3.3, “NDB Cluster Connection Strings”。

有了这些信息，MySQL 服务器可以作为集群中的完整参与者。在这种方式下运行的 **mysqld** 进程通常被称为 SQL 节点。它完全了解所有集群数据节点及其状态，并与所有数据节点建立连接。在这种情况下，它能够使用任何数据节点作为事务协调器，并读取和更新节点数据。

您可以在 **mysql** 客户端中使用 `SHOW PROCESSLIST` 查看 MySQL 服务器是否连接到集群。如果 MySQL 服务器连接到集群，并且您具有 `PROCESS` 权限，则输出的第一行如下所示：

```sql
mysql> SHOW PROCESSLIST \G
*************************** 1\. row ***************************
     Id: 1
   User: system user
   Host:
     db:
Command: Daemon
   Time: 1
  State: Waiting for event from ndbcluster
   Info: NULL
```

重要提示

要参与 NDB Cluster，必须使用*both*选项`--ndbcluster`和`--ndb-connectstring`（或它们在`my.cnf`中的等效选项）启动**mysqld**进程。如果只使用`--ndbcluster`选项启动**mysqld**，或者无法联系到集群，就无法使用`NDB`表，*也无法创建任何新表，无论存储引擎如何*。后一限制是一项安全措施，旨在防止在 SQL 节点未连接到集群时创建与`NDB`表同名的表。如果希望在**mysqld**进程未参与 NDB Cluster 时使用不同的存储引擎创建表，必须在不使用`--ndbcluster`选项的情况下重新启动服务器。
