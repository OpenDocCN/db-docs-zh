> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-debian.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-debian.html)

#### 25.3.1.3 使用 .deb 文件安装 NDB Cluster

本节提供了有关在 Debian 和相关 Linux 发行版（如 Ubuntu）上使用 Oracle 提供的 `.deb` 文件安装 NDB Cluster 的信息。

Oracle 还为 Debian 和其他发行版提供了一个 NDB Cluster APT 软件源。有关说明和其他信息，请参见*使用 APT 软件源安装 MySQL NDB Cluster*。

Oracle 为 32 位和 64 位平台提供了 NDB Cluster 的 `.deb` 安装文件。对于基于 Debian 的系统，只需要一个安装文件。根据适用的 NDB Cluster 版本、Debian 版本和架构，此文件的命名遵循此模式：

```sql
mysql-cluster-gpl-*ndbver*-debian*debianver*-*arch*.deb
```

这里，*`ndbver`* 是 `NDB` 引擎版本号的三部分，*`debianver`* 是 Debian 的主要版本号（`8` 或 `9`），*`arch`* 是 `i686` 或 `x86_64` 中的一个。在接下来的示例中，我们假设您希望在 64 位 Debian 9 系统上安装 NDB 8.0.34；在这种情况下，安装文件名为 `mysql-cluster-gpl-8.0.34-debian9-x86_64.deb-bundle.tar`。

下载适当的 `.deb` 文件后，您可以解压缩它，然后使用 `dpkg` 命令行安装，如下所示：

```sql
$> dpkg -i mysql-cluster-gpl-8.0.34-debian9-i686.deb
```

您也可以像下面这样使用 `dpkg` 进行卸载：

```sql
$> dpkg -r mysql
```

安装文件还应与大多数与 `.deb` 文件一起使用的图形软件包管理器兼容，例如 Gnome 桌面的 `GDebi`。

`.deb` 文件将 NDB Cluster 安装在 `/opt/mysql/server-*`version`*/` 下，其中 *`version`* 是包含的 MySQL 服务器的两部分发布系列版本。对于 NDB 8.0，这始终是 `8.0`。目录布局与通用 Linux 二进制发行版相同（请参见表 2.3，“通用 Unix/Linux 二进制包的 MySQL 安装布局”），唯一的例外是启动脚本和配置文件位于 `support-files` 而不是 `share` 中。所有 NDB Cluster 可执行文件，如 **ndb_mgm**, **ndbd**, 和 **ndb_mgmd**, 都放在 `bin` 目录中。
