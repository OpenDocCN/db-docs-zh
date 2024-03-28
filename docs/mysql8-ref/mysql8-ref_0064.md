# 2.5 在 Linux 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/linux-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/linux-installation.html)

2.5.1 在 Linux 上使用 MySQL Yum 仓库安装 MySQL

2.5.2 在 Linux 上使用 MySQL APT 仓库安装 MySQL

2.5.3 在 Linux 上使用 MySQL SLES 仓库安装 MySQL

2.5.4 在 Linux 上使用来自 Oracle 的 RPM 软件包安装 MySQL

2.5.5 在 Linux 上使用来自 Oracle 的 Debian 软件包安装 MySQL

2.5.6 在 Linux 上使用 Docker 容器部署 MySQL

2.5.7 从本地软件仓库在 Linux 上安装 MySQL

2.5.8 在 Linux 上使用 Juju 安装 MySQL

2.5.9 使用 systemd 管理 MySQL 服务器

Linux 支持多种不同的解决方案来安装 MySQL。我们建议您使用来自 Oracle 的发行版之一，其中提供了多种安装方法：

**表 2.8 Linux 安装方法和信息**

| 类型 | 设置方法 | 附加信息 |
| --- | --- | --- |
| Apt | 启用[MySQL Apt 仓库](https://dev.mysql.com/downloads/repo/apt/) | 文档 |
| Yum | 启用[MySQL Yum 仓库](https://dev.mysql.com/downloads/repo/yum/) | 文档 |
| Zypper | 启用[MySQL SLES 仓库](https://dev.mysql.com/downloads/repo/suse/) | 文档 |
| RPM | [下载](https://dev.mysql.com/downloads/mysql/) 特定软件包 | 文档 |
| DEB | [下载](https://dev.mysql.com/downloads/mysql/) 特定软件包 | 文档 |
| 通用 | [下载](https://dev.mysql.com/downloads/mysql/) 通用软件包 | ��档 |
| Source | 从[源代码](https://dev.mysql.com/downloads/mysql/)编译 | 文档 |
| Docker | 使用[Oracle 容器注册表](https://container-registry.oracle.com/)。您也可以使用[My Oracle Support](https://support.oracle.com/) 来获取 MySQL 企业版。 | 文档 |
| Oracle Unbreakable Linux Network | 使用 ULN 频道 | 文档") |

作为替代方案，您可以使用系统上的软件包管理器从 Linux 发行版的本机软件仓库自动下载并安装 MySQL。这些本机软件包通常落后于当前可用版本。通常也无法安装创新版本，因为这些通常不会在本机仓库中提供。有关使用本机软件包安装程序的更多信息，请参阅 Section 2.5.7, “Installing MySQL on Linux from the Native Software Repositories”。

注意

对于许多 Linux 安装，您希望设置 MySQL 在您的机器启动时自动启动。许多本机软件包安装会为您执行此操作，但对于源码、二进制和 RPM 解决方案，您可能需要单独设置。所需的脚本，**mysql.server**，可以在 MySQL 安装目录下的 `support-files` 目录中或 MySQL 源代码树中找到。您可以将其安装为 `/etc/init.d/mysql` 以实现自动 MySQL 启动和关闭。请参阅 Section 6.3.3, “mysql.server — MySQL Server Startup Script”。
