# 2.3.2 选择安装包

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-choosing-package.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-choosing-package.html)

对于 MySQL 8.0，在 Windows 上安装 MySQL 时，有多种安装包格式可供选择。本节描述的软件包格式包括：

+   MySQL Installer

+   MySQL noinstall ZIP Archives

+   MySQL Docker Images

程序数据库（PDB）文件（文件扩展名为 `pdb`）提供了在出现问题时调试 MySQL 安装的信息。这些文件包含在 MySQL 的 ZIP Archive 发行版中（但不包括 MSI 发行版）。

#### MySQL Installer

该软件包的文件名类似于 `mysql-installer-community-8.0.36.0.msi` 或 `mysql-installer-commercial-8.0.36.0.msi`，并利用 MSIs 自动安装 MySQL 服务器和其他产品。MySQL Installer 下载并应用更新到自身和每个已安装的产品。它还配置已安装的 MySQL 服务器（包括一个沙箱 InnoDB 集群测试设置）和 MySQL Router。MySQL Installer 推荐给大多数用户。

MySQL Installer 可以安装和管理（添加、修改、升级和删除）许多其他 MySQL 产品，包括：

+   应用程序 – MySQL Workbench、MySQL for Visual Studio、MySQL Shell 和 MySQL Router（参见 `dev.mysql.com/doc/mysql-compat-matrix/en/`）

+   连接器 – MySQL Connector/C++、MySQL Connector/NET、Connector/ODBC、MySQL Connector/Python、MySQL Connector/J、MySQL Connector/Node.js

+   文档 – MySQL 手册（PDF 格式）、示例和示例

MySQL Installer 可在所有 MySQL 支持的 Windows 版本上运行（参见 [`www.mysql.com/support/supportedplatforms/database.html`](https://www.mysql.com/support/supportedplatforms/database.html)）。

注意

因为 MySQL Installer 不是 Microsoft Windows 的本机组件，并且依赖于 .NET，所以它不适用于像 Windows Server 的 Server Core 版本这样的最小安装选项。

有关如何使用 MySQL Installer 安装 MySQL 的说明，请参见 Section 2.3.3, “MySQL Installer for Windows”。

#### MySQL noinstall ZIP Archives

这些软件包包含完整 MySQL 服务器安装包中的文件，除了 GUI。这种格式不包括自动安装程序，必须手动安装和配置。

`noinstall` ZIP 归档文件分为两个单独的压缩文件。主包名为`mysql-*`VERSION`*-winx64.zip`。其中包含了在系统上使用 MySQL 所需的组件。可选的 MySQL 测试套件、MySQL 基准套件和调试二进制/信息组件（包括 PDB 文件）位于名为`mysql-*`VERSION`*-winx64-debug-test.zip`的单独压缩文件中。

如果选择安装`noinstall` ZIP 归档文件，请参见 Section 2.3.4, “使用`noinstall` ZIP 归档文件在 Microsoft Windows 上安装 MySQL”。

#### MySQL Docker 镜像

有关在 Windows 平台上使用 Oracle 提供的 MySQL Docker 镜像的信息，请参见 Section 2.5.6.3, “使用 Docker 在 Windows 和其他非 Linux 平台上部署 MySQL”。

警告

Oracle 提供的 MySQL Docker 镜像专为 Linux 平台构建。不支持其他平台，使用 Oracle 提供的 MySQL Docker 镜像在这些平台上运行属于自担风险。
