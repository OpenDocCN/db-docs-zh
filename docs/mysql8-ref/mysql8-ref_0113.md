# 3.11 在 Windows 上升级 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-upgrading.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-upgrading.html)

在 Windows 上升级 MySQL 有两种方法：

+   使用 MySQL Installer

+   使用 Windows ZIP 存档发行版

您选择的方法取决于现有安装是如何进行的。在继续之前，请查看第三章，“*升级 MySQL*”，了解有关升级 MySQL 的其他信息，这些信息不特定于 Windows。

注意

无论您选择哪种方法，在执行升级之前，始终要备份当前的 MySQL 安装。请参阅第 9.2 节，“数据库备份方法”。

不支持非 GA 版本之间的升级（或从非 GA 版本升级到 GA 版本）。非 GA 版本中会发生重大的开发变化，您可能会遇到兼容性问题或启动服务器时出现问题。

注意

MySQL Installer 不支持在*社区*版本和*商业*版本之间的升级。如果您需要此类型的升级，请使用 ZIP 存档方法进行。

### 使用 MySQL Installer 升级 MySQL

当当前服务器安装是使用 MySQL Installer 进行的，并且升级在当前发布系列内时，使用 MySQL Installer 进行升级是最佳方法。MySQL Installer 不支持在发布系列之间的升级，例如从 5.7 升级到 8.0，并且不提供升级指示器来提示您进行升级。有关在发布系列之间升级的说明，请参阅使用 Windows ZIP 发行版升级 MySQL。

要使用 MySQL Installer 进行升级：

1.  启动 MySQL Installer。

1.  从仪表板中，点击 Catalog 下载目录的最新更改。只有在仪表板显示服务器版本号旁边有箭头时，安装的服务器才能升级。

1.  点击升级。现在，所有有更新版本的产品都会显示在列表中。

    注意

    MySQL Installer 取消里程碑版本（预发布）在同一发布系列中的服务器升级选项。此外，它显示警告以指示不支持升级，识别继续的风险，并提供手动执行升级步骤的摘要。您可以重新选择服务器升级并自行承担风险继续进行。

1.  除非您打算此时升级其他产品，否则取消选择除 MySQL 服务器产品之外的所有产品，并点击下一步。

1.  点击“执行”开始下载。下载完成后，点击“下一步”开始升级操作。

    升级到 MySQL 8.0.16 及更高版本可能会显示一个选项，跳过系统表的升级检查和处理。有关此选项的更多信息，请参阅 Important server upgrade conditions。

1.  配置服务器。

### 使用 Windows ZIP 分发升级 MySQL。

使用 Windows ZIP 存档分发执行升级：

1.  从[`dev.mysql.com/downloads/`](https://dev.mysql.com/downloads/)下载最新的 Windows ZIP 存档分发的 MySQL。

1.  如果服务器正在运行，请停止它。如果服务器已安装为服务，请使用命令提示符中的以下命令停止服务：

    ```sql
    C:\> SC STOP *mysqld_service_name*
    ```

    或者，使用**NET STOP *`mysqld_service_name`***。

    如果未将 MySQL 服务器作为服务运行，请使用**mysqladmin**停止它。例如，在从 MySQL 5.7 升级到 8.0 之前，请使用 MySQL 5.7 中的**mysqladmin**如下：

    ```sql
    C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin" -u root shutdown
    ```

    注意

    如果 MySQL `root`用户账户有密码，请使用带有`-p`选项的**mysqladmin**并在提示时输入密码。

1.  解压 ZIP 存档。您可以覆盖现有的 MySQL 安装（通常位于`C:\mysql`），或将其安装到不同的目录，例如`C:\mysql8`。建议覆盖现有安装。

1.  重新启动服务器。例如，如果将 MySQL 作为服务运行，请使用**SC START *`mysqld_service_name`***或**NET START *`mysqld_service_name`***命令，否则直接调用**mysqld**。

1.  在 MySQL 8.0.16 之前，以管理员身份运行**mysql_upgrade**检查您的表，必要时尝试修复它们，并更新授权表（如果已更改），以便利用任何新功能。请参阅 Section 6.4.5, “mysql_upgrade — Check and Upgrade MySQL Tables”。从 MySQL 8.0.16 开始，不再需要此步骤，因为服务器执行了以前由**mysql_upgrade**处理的所有任务。

1.  如果遇到错误，请参阅 Section 2.3.5, “Troubleshooting a Microsoft Windows MySQL Server Installation”。
