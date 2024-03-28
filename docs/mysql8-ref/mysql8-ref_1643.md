> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-source.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-source.html)

#### 25.3.1.4 在 Linux 上从源代码构建 NDB Cluster

本节提供了关于在 Linux 和其他类 Unix 平台上编译 NDB Cluster 的信息。从源代码构建 NDB Cluster 类似于构建标准的 MySQL Server，尽管在一些关键方面有所不同，这些差异在这里讨论。关于从源代码构建 MySQL 的一般信息，请参见 第 2.8 节，“从源代码安装 MySQL”。关于在 Windows 平台上编译 NDB Cluster 的信息，请参见 第 25.3.2.2 节，“在 Windows 上从源代码编译和安装 NDB Cluster”。

构建 MySQL NDB Cluster 8.0 需要使用 MySQL Server 8.0 的源代码。这些源代码可以从 MySQL 下载页面获取，链接为 [`dev.mysql.com/downloads/`](https://dev.mysql.com/downloads/)。存档的源文件应该类似于 `mysql-8.0.34.tar.gz`。你也可以从 GitHub 获取这些源代码，链接为 [`github.com/mysql/mysql-server`](https://github.com/mysql/mysql-server)。

注意

在以前的版本中，从标准的 MySQL Server 源代码构建 NDB Cluster 是不被支持的。在 MySQL 8.0 和 NDB Cluster 8.0 中，情况已经不再是这样了—*这两个产品现在都是从相同的源代码构建的*。

**CMake** 的 `WITH_NDB` 选项会导致管理节点、数据节点和其他 NDB Cluster 程序的二进制文件被构建；它还会导致 **mysqld** 被编译时带有 `NDB` 存储引擎支持。在构建 NDB Cluster 时，这个选项（或在 NDB 8.0.31 之前的版本中，`WITH_NDBCLUSTER`）是必需的。

重要

**WITH_NDB_JAVA** 选项默认启用。这意味着，默认情况下，如果 **CMake** 在您的系统上找不到 Java 的位置，配置过程将失败；如果您不希望启用 Java 和 ClusterJ 支持，您必须显式地通过 `-DWITH_NDB_JAVA=OFF` 配置构建。如果需要，使用 `WITH_CLASSPATH` 提供 Java 类路径。

关于构建 NDB Cluster 的 **CMake** 选项的更多信息，请参见 用于编译 NDB Cluster 的 CMake 选项。

在运行 **make && make install**（或系统的等效命令）之后，结果类似于将预编译的二进制文件解压到相同位置所得到的结果。

**管理节点。** 当从源代码构建并运行默认的 **make install** 时，管理服务器和管理客户端二进制文件（**ndb_mgmd** 和 **ndb_mgm**）可以在 `/usr/local/mysql/bin` 中找到。在管理节点主机上只需要存在 **ndb_mgmd**；但是，在同一主机上也有 **ndb_mgm** 是个好主意。这两个可执行文件都不需要在主机文件系统上的特定位置。

**数据节点。** 数据节点主机上唯一需要的可执行文件是数据节点二进制文件 **ndbd** 或 **ndbmtd**。（例如，**mysqld** 不需要存在于主机机器上。）默认情况下，从源代码构建时，此文件放置在目录 `/usr/local/mysql/bin` 中。对于在多个数据节点主机上安装，只需将 **ndbd** 或 **ndbmtd** 复制到其他主机机器上即可。（这假设所有数据节点主机使用相同的架构和操作系统；否则，您可能需要为每个不同平台单独编译。）数据节点二进制文件不需要在主机文件系统上的任何特定位置，只要位置已知即可。

当从源代码编译 NDB Cluster 时，构建多线程数据节点二进制文件不需要特殊选项。配置构建时使用 `NDB` 存储引擎支持会自动构建 **ndbmtd**；**make install** 将 **ndbmtd** 二进制文件放置在安装的 `bin` 目录中，与 **mysqld**、**ndbd** 和 **ndb_mgm** 一起。

**SQL 节点。** 如果您使用集群支持编译 MySQL，并执行默认安装（使用**make install**作为系统`root`用户），**mysqld**将被放置在`/usr/local/mysql/bin`中。按照 Section 2.8, “从源代码安装 MySQL”中给出的步骤使**mysqld**准备就绪。如果您想要运行多个 SQL 节点，您可以在多台机器上使用相同的**mysqld**可执行文件及其相关的支持文件的副本。最简单的方法是将整个`/usr/local/mysql`目录及其内部的所有目录和文件复制到其他 SQL 节点主机上，然后在每台机器上重复 Section 2.8, “从源代码安装 MySQL”中的步骤。如果您使用非默认的`PREFIX`选项配置构建，则必须相应调整目录。

在 Section 25.3.3, “NDB Cluster 的初始配置”中，我们为示例 NDB Cluster 中的所有节点创建配置文件。
