# 25.4.1 NDB Cluster 的快速测试设置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-quick.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-quick.html)

为了让您熟悉基础知识，我们描述了一个功能性 NDB Cluster 的最简配置。之后，您应该能够根据本章其他相关部分提供的信息设计您所需的设置。

首先，您需要创建一个配置目录，比如`/var/lib/mysql-cluster`，通过以系统`root`用户身份执行以下命令：

```sql
$> mkdir /var/lib/mysql-cluster
```

在这个目录中，创建一个名为`config.ini`的文件，其中包含以下信息。根据您的系统需要，替换`HostName`和`DataDir`的适当值。

```sql
# file "config.ini" - showing minimal setup consisting of 1 data node,
# 1 management server, and 3 MySQL servers.
# The empty default sections are not required, and are shown only for
# the sake of completeness.
# Data nodes must provide a hostname but MySQL Servers are not required
# to do so.
# If you don't know the hostname for your machine, use localhost.
# The DataDir parameter also has a default value, but it is recommended to
# set it explicitly.
# Note: [db], [api], and [mgm] are aliases for [ndbd], [mysqld], and [ndb_mgmd],
# respectively. [db] is deprecated and should not be used in new installations.

[ndbd default]
NoOfReplicas= 1

[mysqld  default]
[ndb_mgmd default]
[tcp default]

[ndb_mgmd]
HostName= myhost.example.com

[ndbd]
HostName= myhost.example.com
DataDir= /var/lib/mysql-cluster

[mysqld]
[mysqld]
[mysqld]
```

您现在可以启动**ndb_mgmd**管理服务器了。默认情况下，它会尝试读取当前工作目录中的`config.ini`文件，因此请切换到文件所在的目录，然后调用**ndb_mgmd**：

```sql
$> cd /var/lib/mysql-cluster
$> ndb_mgmd
```

然后通过运行**ndbd**来启动单个数据节点：

```sql
$> ndbd
```

默认情况下，**ndbd** 在端口 1186 上查找管理服务器的`localhost`。

注意

如果您从二进制 tarball 安装了 MySQL，则必须明确指定**ndb_mgmd**和**ndbd**服务器的路径。（通常可以在`/usr/local/mysql/bin`中找到。）

最后，切换到 MySQL 数据目录（通常为`/var/lib/mysql`或`/usr/local/mysql/data`），确保`my.cnf`文件包含启用 NDB 存储引擎所需的选项：

```sql
[mysqld]
ndbcluster
```

您现在可以像往常一样启动 MySQL 服务器了：

```sql
$> mysqld_safe --user=mysql &
```

等待片刻以确保 MySQL 服务器正常���行。如果看到通知`mysql ended`，请检查服务器的`.err`文件以查找问题所在。

如果到目前为止一切顺利，您现在可以开始使用集群了。连接到服务器并验证`NDBCLUSTER`存储引擎是否已启用：

```sql
$> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1 to server version: 8.0.36

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SHOW ENGINES\G
...
*************************** 12\. row ***************************
Engine: NDBCLUSTER
Support: YES
Comment: Clustered, fault-tolerant, memory-based tables
*************************** 13\. row ***************************
Engine: NDB
Support: YES
Comment: Alias for NDBCLUSTER
...
```

在上面示例输出中显示的行号可能与您的系统上显示的行号不同，这取决于您的服务器配置方式。

尝试创建一个`NDBCLUSTER`表：

```sql
$> mysql
mysql> USE test;
Database changed

mysql> CREATE TABLE ctest (i INT) ENGINE=NDBCLUSTER;
Query OK, 0 rows affected (0.09 sec)

mysql> SHOW CREATE TABLE ctest \G
*************************** 1\. row ***************************
       Table: ctest
Create Table: CREATE TABLE `ctest` (
  `i` int(11) default NULL
) ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 1 row in set (0.00 sec)
```

要检查节点是否正确设置，请启动管理客户端：

```sql
$> ndb_mgm
```

使用管理客户端内的**SHOW**命令获取有关集群状态的报告：

```sql
ndb_mgm> SHOW
Cluster Configuration
---------------------
[ndbd(NDB)]     1 node(s)
id=2    @127.0.0.1  (Version: 8.0.35-ndb-8.0.35, Nodegroup: 0, *)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @127.0.0.1  (Version: 8.0.35-ndb-8.0.35)

[mysqld(API)]   3 node(s)
id=3    @127.0.0.1  (Version: 8.0.35-ndb-8.0.35)
id=4 (not connected, accepting connect from any host)
id=5 (not connected, accepting connect from any host)
```

此时，您已成功设置了一个可工作的 NDB 集群。现在，您可以使用任何使用`ENGINE=NDBCLUSTER`或其别名`ENGINE=NDB`创建的表在集群中存储数据。
