# 2.3.3 Windows 版 MySQL 安装程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-installer.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-installer.html)

2.3.3.1 MySQL 安装程序初始设置

2.3.3.2 使用 MySQL 安装程序设置替代服务器路径

2.3.3.3 使用 MySQL 安装程序进行安装工作流程

2.3.3.4 MySQL 安装程序产品目录和仪表板

2.3.3.5 MySQL 安装程序控制台参考

MySQL 安装程序是一个独立的应用程序，旨在简化在运行 Microsoft Windows 上的 MySQL 产品的安装和配置复杂性。它与以下 MySQL 产品一起下载并支持：

+   MySQL 服务器

    MySQL 安装程序可以在同一主机上同时安装和管理多个独立的 MySQL 服务器实例。例如，MySQL 安装程序可以在同一主机上安装、配置和升级 MySQL 5.7 和 MySQL 8.0 的独立实例。MySQL 安装程序不允许在主要和次要版本号之间进行服务器升级，但允许在一个发布系列内进行升级（例如从 8.0.21 到 8.0.22）。

    注意

    MySQL 安装程序无法在同一主机上安装 *Community* 和 *Commercial* 版本的 MySQL 服务器。如果您需要在同一主机上安装两个版本，请考虑使用 ZIP 存档分发来安装其中一个版本。

+   MySQL 应用程序

    MySQL Workbench、MySQL Shell 和 MySQL Router。

+   MySQL 连接器（直至 MySQL 8.0.33）

    MySQL Connector/NET、MySQL Connector/Python、MySQL Connector/ODBC、MySQL Connector/J 和 MySQL Connector/C++。要安装 MySQL Connector/Node.js，请参阅[`dev.mysql.com/downloads/connector/nodejs/`](https://dev.mysql.com/downloads/connector/nodejs/)。

    注意

    要安装更新的 MySQL 连接器，请访问 https://dev.mysql.com/downloads/。

#### 安装要求

MySQL 安装程序需要 Microsoft .NET Framework 4.5.2 或更高版本。如果主机计算机上未安装此版本，请访问[微软网站](https://www.microsoft.com/en-us/download/details.aspx?id=42643)下载。

在成功安装后调用 MySQL 安装程序：

1.  右键单击 Windows 开始，选择运行，然后单击浏览。导航至`Program Files (x86) > MySQL > MySQL Installer for Windows`以打开程序文件夹。

1.  选择以下文件之一：

    +   `MySQLInstaller.exe` 以打开图形应用程序。

    +   `MySQLInstallerConsole.exe` 以打开命令行应用程序。

1.  单击打开，然后在运行窗口中单击确定。如果提示允许应用程序对设备进行更改，请选择`是`。

每次调用 MySQL 安装程序时，初始化过程会检查是否有互联网连接，并在找不到互联网访问时（且离线模式已禁用）提示您启用离线模式。选择`是`以在没有互联网连接功能的情况下运行 MySQL 安装程序。启用离线模式时，MySQL 产品的可用性仅限于在启用离线模式时在产品缓存中的产品。要下载 MySQL 产品，请单击仪表板上显示的离线模式禁用快速操作。

下载最新 MySQL 产品的元数据清单需要互联网连接，这些产品不是完整捆绑包的一部分。当您首次启动应用程序时，MySQL 安装程序会尝试下载清单，然后在可配置的间隔时间内定期下载（请参阅 MySQL 安装程序选项）。或者，您可以通过在 MySQL 安装程序仪表板中单击目录来手动检索更新的清单。

注意

如果首次或后续清单下载失败，将记录错误，并且在会话期间可能会限制您对 MySQL 产品的访问。MySQL 安装程序会在每次启动时尝试下载清单，直到初始清单结构更新为止。如需帮助查找产品，请参阅查找要安装的产品。

#### MySQL 安装程序社区版本

从[`dev.mysql.com/downloads/installer/`](https://dev.mysql.com/downloads/installer/)下载软件以安装 Windows 的所有 MySQL 产品的社区版本。选择以下 MySQL 安装程序包选项：

+   *Web*：仅包含 MySQL 安装程序和配置文件。Web 包选项仅下载您选择安装的 MySQL 产品，但每次下载都需要互联网连接。该文件的大小约为 2 MB。文件名的形式为`mysql-installer-community-`web`-*`VERSION`*.*`N`*.msi`，其中*`VERSION`*是 MySQL 服务器版本号，例如 8.0，`N`是包号，从 0 开始。

+   *完整或当前捆绑包*：捆绑了所有 Windows 的 MySQL 产品（包括 MySQL 服务器）。文件大小超过 300 MB，名称的形式为`mysql-installer-community-*`VERSION`*.*`N`*.msi`，其中*`VERSION`*是 MySQL Server 版本号，例如 8.0，`N`是包号，从 0 开始。

#### MySQL 安装程序商业版本

从[`edelivery.oracle.com/`](https://edelivery.oracle.com/)下载软件，以安装 MySQL 产品的商业版（标准版或企业版）用于 Windows。如果您已登录到您的我的 Oracle 支持（MOS）账户，商业版包括社区版中所有当前和以前的 GA 版本，但不包括开发里程碑版本。当您未登录时，您只能看到您已经下载的捆绑产品列表。

商业版还包括以下产品：

+   工作台 SE/EE

+   MySQL 企业备份

+   MySQL 企业防火墙

商业版与您的 MOS 账户集成。有关知识库内容和补丁，请参见[我的 Oracle 支持](https://support.oracle.com/)。
