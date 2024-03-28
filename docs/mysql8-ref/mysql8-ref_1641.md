> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-rpm.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-rpm.html)

#### 25.3.1.2 从 RPM 安装 NDB Cluster

本节涵盖了使用 Oracle 提供的 RPM 软件包安装每种类型的 NDB Cluster 8.0 节点所需的正确可执行文件的步骤。

作为本节描述的方法的替代方案，Oracle 为 NDB Cluster 提供了与许多常见 Linux 发行版兼容的 MySQL 存储库。列出了两个存储库，适用于基于 RPM 的发行版：

+   对于使用**yum**或**dnf**的发行版，您可以使用 MySQL Yum Repository for NDB Cluster。有关说明和其他信息，请参见*使用 Yum 存储库安装 MySQL NDB Cluster*。

+   对于 SLES，您可以使用 MySQL SLES Repository for NDB Cluster。有关说明和其他信息，请参见*使用 SLES 存储库安装 MySQL NDB Cluster*。

32 位和 64 位 Linux 平台均提供 RPM。这些 RPM 的文件名遵循以下模式：

```sql
mysql-cluster-community-data-node-8.0.34-1.el7.x86_64.rpm

mysql-cluster-*license*-*component*-*ver*-*rev*.*distro*.*arch*.rpm

    *license*:= {commercial | community}

    *component*: {management-server | data-node | server | client | *other—see text*}

    *ver*: *major*.*minor*.*release*

    *rev*: *major*[.*minor*]

    *distro*: {el6 | el7 | sles12}

    *arch*: {i686 | x86_64}
```

*`license`*指示 RPM 是否属于 NDB Cluster 的商业版或社区版。在本节的其余部分中，我们假设您正在安装社区版。

*`component`*的可能值及其描述可在以下表中找到：

**表 25.6 NDB Cluster RPM 分发的组件**

| 组件 | 描述 |
| --- | --- |
| `auto-installer` (已弃用) | NDB Cluster 自动安装程序；请参见第 25.3.8 节，“NDB Cluster 自动安装程序（不再支持）”")，了解用法 |
| `client` | MySQL 和`NDB`客户端程序；包括**mysql**客户端，**ndb_mgm**客户端和其他客户端工具 |
| `common` | MySQL 服务器所需的字符集和错误消息信息 |
| `data-node` | **ndbd**和**ndbmtd**")数据节点二进制文件 |
| `devel` | MySQL 客户端开发所需的头文件和库文件 |
| `embedded` | 嵌入式 MySQL 服务器 |
| `embedded-compat` | 向后兼容的嵌入式 MySQL 服务器 |
| `embedded-devel` | 用于开发嵌入式 MySQL 应用程序的头文件和库文件 |
| `java` | 用于支持 ClusterJ 应用程序的 JAR 文件 |
| `libs` | MySQL 客户端库 |
| `libs-compat` | 向后兼容的 MySQL 客户端库 |
| `management-server` | NDB 集群管理服务器 (**ndb_mgmd**) |
| `memcached` | 支持 `ndbmemcache` 所需的文件 |
| `minimal-debuginfo` | 用于 package server-minimal 的调试信息；在开发使用此 package 的应用程序或调试此 package 时非常有用 |
| `ndbclient` | 用于运行 NDB API 和 MGM API 应用程序的 NDB 客户端库 (`libndbclient`) |
| `ndbclient-devel` | 用于开发 NDB API 和 MGM API 应用程序所需的头文件和其他文件 |
| `nodejs` | 用于设置 NDB 集群的 Node.JS 支持所需的文件 |
| `server` | 包含 `NDB` 存储引擎支持的 MySQL 服务器 (**mysqld**)，以及相关的 MySQL 服务器程序 |
| `server-minimal` | 用于 NDB 和相关工具的 MySQL 服务器的最小安装 |
| `test` | **mysqltest**，其他 MySQL 测试程序和支持文件 |
| 组件 | 描述 |

也可以获得给定平台和架构的所有 NDB 集群 RPM 的单个捆绑包（`.tar` 文件）。此文件的名称遵循此处显示的模式：

```sql
mysql-cluster-*license*-*ver*-*rev*.*distro*.*arch*.rpm-bundle.tar
```

您可以使用 **tar** 或您喜欢的提取存档工具从此文件中提取单独的 RPM 文件。

安装三种主要类型的 NDB 集群节点所需的组件列在以下列表中：

+   *管理节点*: `management-server`

+   *数据节点*: `data-node`

+   *SQL 节点*: `server` 和 `common`

另外，应安装 `client` RPM 以在至少一个管理节点上提供 **ndb_mgm** 管理客户端。您可能还希望在 SQL 节点上安装它，以便在这些节点上提供 **mysql** 和其他 MySQL 客户端程序。我们稍后在本节讨论按类型安装节点。

*`ver`* 表示以 8.0.*`x`* 格式显示的三部分 `NDB` 存储引擎版本号，示例中显示为 `8.0.34`。`rev` 提供了以 *`major`*.*`minor`* 格式的 RPM 修订号。在本节中显示的示例中，我们使用 `1.1` 作为此值。

*`distro`*（Linux 发行版）是 `rhel5`（Oracle Linux 5，Red Hat Enterprise Linux 4 和 5），`el6`（Oracle Linux 6，Red Hat Enterprise Linux 6），`el7`（Oracle Linux 7，Red Hat Enterprise Linux 7）或 `sles12`（SUSE Enterprise Linux 12）之一。在本节的示例中，我们假设主机运行 Oracle Linux 7，Red Hat Enterprise Linux 7 或等效的 (`el7`)。

*`arch`* 对于 32 位 RPMs 是 `i686`，对于 64 位版本是 `x86_64`。在这里展示的示例中，我们假设是 64 位平台。

RPM 文件名中的 NDB 集群版本号（此处显示为`8.0.34`）可能根据您实际使用的版本而变化。*非常重要的是要安装的所有集群 RPM 具有相同的版本号*。架构也应适合要安装 RPM 的机器；特别要记住，64 位 RPM（`x86_64`）不能与 32 位操作系统一起使用（对于后者使用`i686`）。

**数据节点。** 在要托管 NDB 集群数据节点的计算机上，只需安装`data-node` RPM。为此，将此 RPM 复制到数据节点主机，并以系统 root 用户身份运行以下命令，根据需要替换从 MySQL 网站下载的 RPM 的名称：

```sql
$> rpm -Uhv mysql-cluster-community-data-node-8.0.34-1.el7.x86_64.rpm
```

这将在`/usr/sbin`中安装**ndbd**和**ndbmtd**数据节点二进制文件。这两者中的任何一个都可以用于在此主机上运行数据节点进程。

**SQL 节点。** 将`server`和`common` RPM 复制到每台用于托管 NDB 集群 SQL 节点的机器上（`server` 需要`common`）。以系统 root 用户身份执行以下命令安装`server` RPM，根据需要替换从 MySQL 网站下载的 RPM 的名称：

```sql
$> rpm -Uhv mysql-cluster-community-server-8.0.34-1.el7.x86_64.rpm
```

这将在`/usr/sbin`目录中安装 MySQL 服务器二进制文件（**mysqld**），支持`NDB`存储引擎。它还安装了所有必需的 MySQL 服务器支持文件和有用的 MySQL 服务器程序，包括**mysql.server**和**mysqld_safe**启动脚本（分别位于`/usr/share/mysql`和`/usr/bin`）。RPM 安装程序应自动处理一般配置问题（例如自动创建`mysql`用户和组，如果需要）。

重要

您必须使用为 NDB 集群发布的这些 RPM 版本；为标准 MySQL 服务器发布的版本不支持`NDB`存储引擎。

要管理 SQL 节点（MySQL 服务器），您还应安装`client` RPM，如下所示：

```sql
$> rpm -Uhv mysql-cluster-community-client-8.0.34-1.el7.x86_64.rpm
```

这将安装**mysql**客户端和其他 MySQL 客户端程序，如**mysqladmin**和**mysqldump**到`/usr/bin`。

**管理节点。** 要安装 NDB 集群管理服务器，只需使用`management-server` RPM。将此 RPM 复制到打算托管管理节点的计算机上，然后以系统根用户身份运行以下命令进行安装（根据需要替换从 MySQL 网站下载的`management-server` RPM 的名称）：

```sql
$> rpm -Uhv mysql-cluster-community-management-server-8.0.34-1.el7.x86_64.rpm
```

此 RPM 在`/usr/sbin`目录中安装管理服务器二进制文件**ndb_mgmd**。虽然这是实际运行管理节点所需的唯一程序，但也最好同时拥有**ndb_mgm** NDB 集群管理客户端。您可以通过按照先前描述的方式安装`client` RPM 来获取此程序，以及其他`NDB`客户端程序，如**ndb_desc**和**ndb_config**。

有关使用 Oracle 提供的 RPM 包在 Linux 上安装 MySQL 的一般信息，请参阅 Section 2.5.4, “Installing MySQL on Linux Using RPM Packages from Oracle”。

从 RPM 安装后，仍然需要配置集群；请参阅 Section 25.3.3, “Initial Configuration of NDB Cluster”，获取相关信息。

*非常重要的是要安装的所有集群 RPM 具有相同的版本号*。*`架构`*指定也应适合要安装 RPM 的计算机；特别要记住 64 位 RPM 不能与 32 位操作系统一起使用。

**数据节点。** 在打算托管集群数据节点的计算机上，只需安装`server` RPM。为此，将此 RPM 复制到数据节点主机上，并以系统根用户身份运行以下命令，根据需要替换从 MySQL 网站下载的 RPM 的名称：

```sql
$> rpm -Uhv MySQL-Cluster-server-gpl-8.0.34-1.sles11.i386.rpm
```

尽管这会安装所有 NDB 集群二进制文件，但实际上只需要运行 NDB 集群数据节点的程序**ndbd**或**ndbmtd**")（都在`/usr/sbin`中）。

**SQL 节点。** 在每台用于托管集群 SQL 节点的计算机上，以系统根用户身份执行以下命令安装`server` RPM，根据需要替换从 MySQL 网站下载的 RPM 的名称：

```sql
$> rpm -Uhv MySQL-Cluster-server-gpl-8.0.34-1.sles11.i386.rpm
```

这将在`/usr/sbin`目录中安装带有`NDB`存储引擎支持的 MySQL 服务器二进制文件（**mysqld**），以及所有所需的 MySQL 服务器支持文件。它还安装了**mysql.server**和**mysqld_safe**启动脚本（分别位于`/usr/share/mysql`和`/usr/bin`）。RPM 安装程序应自动处理一般配置问题（例如，如有需要，创建`mysql`用户和组）。

要管理 SQL 节点（MySQL 服务器），您还应安装`client` RPM，如下所示：

```sql
$> rpm -Uhv MySQL-Cluster-client-gpl-8.0.34-1.sles11.i386.rpm
```

这将安装**mysql**客户端程序。

**管理节点。** 要安装 NDB Cluster 管理服务器，只需使用`server` RPM。将此 RPM 复制到打算托管管理节点的计算机上，然后以系统 root 用户身份运行以下命令进行安装（根据需要替换从 MySQL 网站下载的`server` RPM 的名称）：

```sql
$> rpm -Uhv MySQL-Cluster-server-gpl-8.0.34-1.sles11.i386.rpm
```

尽管此 RPM 安装了许多其他文件，但实际上只需要管理服务器二进制文件**ndb_mgmd**（位于`/usr/sbin`目录）来运行管理节点。`server` RPM 还安装了**ndb_mgm**，`NDB`管理客户端。

有关使用 Oracle 提供的 RPM 包在 Linux 上安装 MySQL 的一般信息，请参见第 2.5.4 节，“使用 Oracle 提供的 RPM 包在 Linux 上安装 MySQL”。有关所需的安装后配置信息，请参见第 25.3.3 节，“NDB Cluster 的初始配置”。
