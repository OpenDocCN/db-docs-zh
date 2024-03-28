# 25.3.1 在 Linux 上安装 NDB 集群

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux.html)

25.3.1.1 在 Linux 上安装 NDB 集群二进制发行版

25.3.1.2 从 RPM 安装 NDB 集群

25.3.1.3 使用 .deb 文件安装 NDB 集群

25.3.1.4 在 Linux 上从源代码构建 NDB 集群

25.3.1.5 使用 Docker 容器部署 NDB 集群

本节涵盖了在 Linux 和其他类 Unix 操作系统上安装 NDB 集群的方法。虽然接下来的几节涉及 Linux 操作系统，但那里给出的说明和步骤应该很容易适应其他支持的类 Unix 平台。有关 Windows 系统的手动安装和设置说明，请参阅 第 25.3.2 节“在 Windows 上安装 NDB 集群”。

每台 NDB 集群主机必须安装正确的可执行程序。运行 SQL 节点的主机必须安装 MySQL 服务器二进制文件（**mysqld**）。管理节点需要管理服务器守护程序（**ndb_mgmd**）；数据节点需要数据节点守护程序（**ndbd**或 **ndbmtd**）。在管理节点主机和数据节点主机上安装 MySQL 服务器二进制文件并非必需。建议在管理服务器主机上也安装管理客户端（**ndb_mgm**）。

在 Linux 上安装 NDB 集群可以使用来自 Oracle 的预编译二进制文件（以 .tar.gz 存档文件下载）、RPM 包（也可从 Oracle 获取）或源代码。这三种安装方法在接下来的章节中都有描述。

无论使用何种方法，在安装 NDB 集群二进制文件后，仍然需要为所有集群节点创建配置文件，然后才能启动集群。请参阅 第 25.3.3 节“NDB 集群的初始配置”。
