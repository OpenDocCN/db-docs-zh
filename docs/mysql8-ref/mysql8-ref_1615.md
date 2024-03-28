# 25.1 常规信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-general-info.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-general-info.html)

MySQL NDB Cluster 使用带有`NDB`存储引擎的 MySQL 服务器。 Oracle 构建的标准 MySQL Server 8.0 二进制版本不包含对`NDB`存储引擎的支持。 相反，Oracle 的 NDB Cluster 二进制版本用户应升级到支持平台上最新的 NDB Cluster 二进制版本 - 这些版本包括应该适用于大多数 Linux 发行版的 RPM。 从源代码构建的 NDB Cluster 8.0 用户应使用为 MySQL 8.0 提供 NDB 支持所需的选项构建提供的源代码。 （可以在本节后面列出的位置获取源代码的地点。）

重要

MySQL NDB Cluster 不支持 InnoDB Cluster，必须使用带有`InnoDB`存储引擎的 MySQL Server 8.0 部署，以及未包含在 NDB Cluster 发行版中的其他应用程序。 MySQL Server 8.0 二进制版本不能与 MySQL NDB Cluster 一起使用。 有关部署和使用 InnoDB Cluster 的更多信息，请参阅 MySQL AdminAPI。 第 25.2.6 节，“MySQL Server 使用 InnoDB 与 NDB Cluster 比较”，讨论了`NDB`和`InnoDB`存储引擎之间的差异。

**支持的平台。** NDB Cluster 目前在许多平台上可用并受支持。 有关特定操作系统版本、操作系统发行版和硬件平台组合的确切支持级别，请参阅[`www.mysql.com/support/supportedplatforms/cluster.html`](https://www.mysql.com/support/supportedplatforms/cluster.html)。

**可用性。** NDB Cluster 二进制和源代码包可在[`dev.mysql.com/downloads/cluster/`](https://dev.mysql.com/downloads/cluster/)上的支持平台上获得。

**NDB Cluster 发行版本号。** NDB 8.0 遵循与 MySQL Server 8.0 系列发布相同的发布模式，从 MySQL 8.0.13 和 MySQL NDB Cluster 8.0.13 开始。 在本 *手册* 和其他 MySQL 文档中，我们使用以“NDB”开头的版本号来标识这些以及后续的 NDB Cluster 发行版，这个版本号是 NDB 8.0 发行版中使用的`NDBCLUSTER`存储引擎的版本号，并且与基于的 MySQL 8.0 服务器版本相同 NDB Cluster 8.0 发行版。

**NDB Cluster 软件中使用的版本字符串。** MySQL NDB Cluster 发行版附带的**mysql**客户端显示的版本字符串使用此格式：

```sql
mysql-*mysql_server_version*-cluster
```

*`mysql_server_version`* 代表 NDB Cluster 发行版所基于的 MySQL Server 版本。对于所有 NDB Cluster 8.0 发行版，这是 `8.0.*`n`*`，其中 *`n`* 是发行号。使用 `-DWITH_NDB` 或等效选项从源代码构建时，会在版本字符串中添加 `-cluster` 后缀。（参见 第 25.3.1.4 节，“在 Linux 上从源代码构建 NDB Cluster”，以及 第 25.3.2.2 节，“在 Windows 上从源代码编译和安装 NDB Cluster”。）你可以在 **mysql** 客户端中看到这种格式的使用，如下所示：

```sql
$> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 8.0.35-cluster Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SELECT VERSION()\G
*************************** 1\. row ***************************
VERSION(): 8.0.35-cluster 1 row in set (0.00 sec)
```

使用 MySQL 8.0 的第一个正式发布的 NDB Cluster 版本是 NDB 8.0.19，使用 MySQL 8.0.19。

其他未包含在 MySQL 8.0 发行版中的 NDB Cluster 程序显示的版本字符串采用以下格式：

```sql
mysql-*mysql_server_version* ndb-*ndb_engine_version*
```

*`mysql_server_version`* 代表 NDB Cluster 发行版所基于的 MySQL Server 版本。对于所有 NDB Cluster 8.0 发行版，这是 `8.0.*`n`*`，其中 *`n`* 是发行号。 *`ndb_engine_version`* 是此 NDB Cluster 软件版本使用的 `NDB` 存储引擎的版本。对于所有 NDB 8.0 发行版，此数字与 MySQL Server 版本相同。你可以在 **ndb_mgm** 客户端的 `SHOW` 命令输出中看到这种格式的使用，如下所示：

```sql
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @10.0.10.6  (mysql-8.0.35 ndb-8.0.35, Nodegroup: 0, *)
id=2    @10.0.10.8  (mysql-8.0.35 ndb-8.0.35, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=3    @10.0.10.2  (mysql-8.0.35 ndb-8.0.35)

[mysqld(API)]   2 node(s)
id=4    @10.0.10.10  (mysql-8.0.35 ndb-8.0.35)
id=5 (not connected, accepting connect from any host)
```

**与标准 MySQL 8.0 版本的兼容性。** 虽然许多标准 MySQL 架构和应用程序可以在 NDB Cluster 中运行，但未经修改的应用程序和数据库架构可能会略有不兼容或在使用 NDB Cluster 时性能不佳（参见《第 25.2.7 节，“NDB Cluster 的已知限制”》）。大多数问题都可以克服，但这也意味着您很难将当前使用例如 `MyISAM` 或 `InnoDB` 的现有应用程序数据存储切换到使用 `NDB` 存储引擎，而不考虑架构、查询和应用程序可能发生变化的可能性。没有使用 `NDB` 支持编译的 **mysqld**（即没有使用 `-DWITH_NDB` 或 `-DWITH_NDBCLUSTER_STORAGE_ENGINE` 构建的）无法作为使用它构建的 **mysqld** 的直接替代品。

**NDB Cluster 开发源码树。** NDB Cluster 开发树也可以从 [`github.com/mysql/mysql-server`](https://github.com/mysql/mysql-server) 访问。

在 [`github.com/mysql/mysql-server`](https://github.com/mysql/mysql-server) 维护的 NDB Cluster 开发源代码受 GPL 许可。有关使用 Git 获取 MySQL 源代码并自行构建的信息，请参阅《第 2.8.5 节，“使用开发源码树安装 MySQL”》。

注意

与 MySQL Server 8.0 一样，NDB Cluster 8.0 版本是使用 **CMake** 构建的。

NDB Cluster 8.0 从 NDB 8.0.19 开始作为正式版本发布，建议用于新部署。NDB Cluster 7.6 和 7.5 是之前的正式版本，仍在生产中得到支持；有关 NDB Cluster 7.6 的信息，请参阅《NDB Cluster 7.6 有什么新特性》。有关 NDB Cluster 7.5 的类似信息，请参阅《NDB Cluster 7.5 有什么新特性》。NDB Cluster 7.4 和 7.3 是之前的正式版本，不再维护。我们建议新的生产部署使用 MySQL NDB Cluster 8.0。

本章内容可能会随着 NDB Cluster 的不断发展而进行修订。有关 NDB Cluster 的其他信息可以在 MySQL 网站上找到，网址为 [`www.mysql.com/products/cluster/`](http://www.mysql.com/products/cluster/)。

**额外资源。** 关于 NDB Cluster 的更多信息可以在以下地方找到：

+   欲了解关于 NDB Cluster 的一些常见问题的答案，请参阅 Section A.10, “MySQL 8.0 FAQ: NDB Cluster”。

+   NDB Cluster 论坛：[`forums.mysql.com/list.php?25`](https://forums.mysql.com/list.php?25)。

+   许多 NDB Cluster 的用户和开发者在他们的博客中分享了他们与 NDB Cluster 的经验，并通过[PlanetMySQL](http://www.planetmysql.org/)提供这些内容的订阅。
