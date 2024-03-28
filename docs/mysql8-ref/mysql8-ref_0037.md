# 2.3 在 Microsoft Windows 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-installation.html)

2.3.1 Microsoft Windows 上的 MySQL 安装布局

2.3.2 选择安装包

2.3.3 Windows 上的 MySQL 安装程序

2.3.4 使用`noinstall` ZIP 存档在 Microsoft Windows 上安装 MySQL

2.3.5 解决 Microsoft Windows 上的 MySQL 服务器安装问题

2.3.6 Windows 后安装程序

2.3.7 Windows 平台限制

重要提示

MySQL 8.0 Server 在 Windows 平台上运行需要 Microsoft Visual C++ 2019 Redistributable Package。用户在安装服务器之前应确保该软件包已在系统上安装。该软件包可在[Microsoft Download Center](http://www.microsoft.com/en-us/download/default.aspx)上获得。此外，MySQL 调试二进制文件需要安装 Visual Studio 2019。

MySQL 仅适用于 Microsoft Windows 64 位操作系统。有关支持的 Windows 平台信息，请参见[`www.mysql.com/support/supportedplatforms/database.html`](https://www.mysql.com/support/supportedplatforms/database.html)。

在 Microsoft Windows 上安装 MySQL 有不同的方法。

### MySQL 安装程序方法

最简单且推荐的方法是下载 MySQL 安装程序（用于 Windows），并让其安装和配置特定版本的 MySQL Server 如下：

1.  从[`dev.mysql.com/downloads/installer/`](https://dev.mysql.com/downloads/installer/)下载 MySQL 安装程序并执行。

    注意

    与标准 MySQL 安装程序不同，较小的`web-community`版本不捆绑任何 MySQL 应用程序，而只下载您选择安装的 MySQL 产品。

1.  确定用于 MySQL 产品初始安装的设置类型。例如：

    +   开发者默认：提供了一个包括所选版本的 MySQL Server 和其他与 MySQL 开发相关的 MySQL 工具的设置类型，如 MySQL Workbench。

    +   仅服务器：为所选版本的 MySQL Server 提供一个设置，不包括其他产品。

    +   自定义：允许您选择任何版本的 MySQL Server 和其他 MySQL 产品。

1.  安装服务器实例（和产品），然后按照屏幕上的说明开始服务器配置。有关每个单独步骤的更多信息，请参见 Section 2.3.3.3.1, “使用 MySQL 安装程序配置 MySQL 服务器”。

MySQL 现在已安装。如果您将 MySQL 配置为服务，则 Windows 每次重新启动系统时都会自动启动 MySQL 服务器。此外，此过程会在本地主机上安装 MySQL Installer 应用程序，您以后可以使用它来升级或重新配置 MySQL 服务器。

注意

如果您在系统上安装了 MySQL Workbench，请考虑使用它来检查您的新 MySQL 服务器连接。默认情况下，安装 MySQL 后程序会自动启动。

### 附加安装信息

可以将 MySQL 作为标准应用程序或 Windows 服务运行。通过使用服务，您可以通过标准的 Windows 服务管理工具监视和控制服务器的运行。有关更多信息，请参见 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

为了适应`RESTART`语句，当作为服务或独立运行时，MySQL 服务器会进行分叉，以便监视进程监督服务器进程。在这种情况下，会有两个**mysqld**进程。如果不需要`RESTART`功能，则可以使用`--no-monitor`选项启动服务器。请参见 Section 15.7.8.8, “RESTART Statement”。

通常，您应该使用具有管理员权限的帐户在 Windows 上安装 MySQL。否则，您可能会在某些操作（如编辑`PATH`环境变量或访问**服务控制管理器**）中遇到问题。安装后，MySQL 不需��使用具有管理员权限的用户执行。

有关在 Windows 平台上使用 MySQL 的限制列表，请参见 Section 2.3.7, “Windows Platform Restrictions”。

除了 MySQL Server 软件包外，您可能需要或希望使用其他组件来与您的应用程序或开发环境一起使用 MySQL。这些组件包括但不限于：

+   要使用 ODBC 连接到 MySQL 服务器，您必须拥有 Connector/ODBC 驱动程序。有关更多信息，包括安装和配置说明，请参见 MySQL Connector/ODBC Developer Guide。

    注意

    MySQL Installer 会为您安装和配置 Connector/ODBC。

+   要在.NET 应用程序中使用 MySQL 服务器，您必须拥有 Connector/NET 驱动程序。有关更多信息，包括安装和配置说明，请参见 MySQL Connector/NET Developer Guide。

    注意

    MySQL Installer 会为您安装和配置 MySQL Connector/NET。

Windows 上的 MySQL 发行版可以从[`dev.mysql.com/downloads/`](https://dev.mysql.com/downloads/)下载。请参见 Section 2.1.3, “How to Get MySQL”。

Windows 版本的 MySQL 有几种分发格式，详细信息如下。一般来说，您应该使用 MySQL Installer。它包含比旧的 MSI 更多的功能和 MySQL 产品，比压缩文件更简单易用，并且您无需额外的工具即可运行 MySQL。MySQL Installer 自动安装 MySQL Server 和其他 MySQL 产品，创建一个选项文件，启动服务器，并允许您创建默认用户帐户。有关选择软件包的更多信息，请参见 第 2.3.2 节，“选择安装软件包”。

+   MySQL Installer 分发包括 MySQL Server 和其他 MySQL 产品，包括 MySQL Workbench 和 MySQL for Visual Studio。MySQL Installer 还可用于将来升级这些产品（请参见 `dev.mysql.com/doc/mysql-compat-matrix/en/`）。

    有关使用 MySQL Installer 安装 MySQL 的说明，请参见 第 2.3.3 节，“Windows 上的 MySQL Installer”。

+   标准二进制分发（打包为压缩文件）包含您解压缩到所选位置的所有必要文件。此软件包包含完整 Windows MSI 安装程序软件包中的所有文件，但不包括安装程序。

    有关使用压缩文件安装 MySQL 的说明，请参见 第 2.3.4 节，“使用 `noinstall` ZIP 存档在 Microsoft Windows 上安装 MySQL”。

+   源码分发格式包含所有代码和支持文件，用于使用 Visual Studio 编译器系统构建可执行文件。

    构建 MySQL 源码的 Windows 指南，请参见 第 2.8 节，“从源码安装 MySQL”。

### Windows 上的 MySQL 注意事项

+   **大表支持**

    如果您需要大于 4GB 的表，请在 NTFS 或更新的文件系统上安装 MySQL。在创建表时不要忘记使用 `MAX_ROWS` 和 `AVG_ROW_LENGTH`。请参见 第 15.1.20 节，“CREATE TABLE 语句”。

+   **MySQL 和病毒检查软件**

    在包含 MySQL 数据和临时表的目录上运行诺顿/赛门铁克防病毒软件可能会导致问题，这会影响 MySQL 的性能，同时防病毒软件可能会错误地将文件内容识别为垃圾邮件。这是由于防病毒软件使用的指纹机制以及 MySQL 快速更新不同文件的方式，这可能被识别为潜在的安全风险。

    安装完 MySQL 服务器后，建议在用于存储 MySQL 表数据的主目录（`datadir`）上禁用病毒扫描。通常，病毒扫描软件内置了一个功能，可以忽略特定目录。

    另外，默认情况下，MySQL 会在标准 Windows 临时目录中创建临时文件。为了防止临时文件被扫描，配置一个单独的临时目录用于 MySQL 临时文件，并将该目录添加到病毒扫描排除列表中。要做到这一点，向你的`my.ini`配置文件添加一个`tmpdir`参数的配置选项。更多信息，请参见 Section 2.3.4.2, “Creating an Option File”。
