# 第二章 安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/installing.html`](https://dev.mysql.com/doc/refman/8.0/en/installing.html)

**目录**

2.1 一般安装指导

2.1.1 支持的平台

2.1.2 安装哪个 MySQL 版本和发行版

2.1.3 如何获取 MySQL

2.1.4 使用 MD5 校验和或 GnuPG 验证软件包完整性

2.1.5 安装布局

2.1.6 编译器特定的构建特性

2.2 在 Unix/Linux 上使用通用二进制文件安装 MySQL

2.3 在 Microsoft Windows 上安装 MySQL

2.3.1 Microsoft Windows 上的 MySQL 安装布局

2.3.2 选择安装包

2.3.3 Windows 的 MySQL 安装程序

2.3.4 使用 `noinstall` ZIP 存档在 Microsoft Windows 上安装 MySQL

2.3.5 解决 Microsoft Windows 上 MySQL 服务器安装问题

2.3.6 Windows 安装后的程序

2.3.7 Windows 平台限制

2.4 在 macOS 上安装 MySQL

2.4.1 在 macOS 上安装 MySQL 的一般注意事项

2.4.2 使用本地软件包在 macOS 上安装 MySQL

2.4.3 安装和使用 MySQL 启动守护程序

2.4.4 安装和使用 MySQL 首选项窗格

2.5 在 Linux 上安装 MySQL

2.5.1 在 Linux 上使用 MySQL Yum 仓库安装 MySQL

2.5.2 在 Linux 上使用 MySQL APT 仓库安装 MySQL

2.5.3 在 Linux 上使用 MySQL SLES 仓库安装 MySQL

2.5.4 使用 Oracle 的 RPM 软件包在 Linux 上安装 MySQL

2.5.5 在 Linux 上使用 Oracle Debian 软件包安装 MySQL

2.5.6 使用 Docker 容器在 Linux 上部署 MySQL

2.5.7 从本地软件仓库在 Linux 上安装 MySQL

2.5.8 使用 Juju 在 Linux 上安装 MySQL

2.5.9 使用 systemd 管理 MySQL 服务器

2.6 使用 Unbreakable Linux Network (ULN) 安装 MySQL

2.7 在 Solaris 上安装 MySQL

2.7.1 使用 Solaris PKG 在 Solaris 上安装 MySQL

2.8 从源代码安装 MySQL

2.8.1 源码安装方法

2.8.2 源码安装先决条件

2.8.3 源码安装 MySQL 布局

2.8.4 使用标准源码分发安装 MySQL

2.8.5 使用开发源树安装 MySQL

2.8.6 配置 SSL 库支持

2.8.7 MySQL 源码配置选项

2.8.8 处理编译 MySQL 时的问题

2.8.9 MySQL 配置和第三方工具

2.8.10 生成 MySQL Doxygen 文档内容

2.9 安装后设置和测试

2.9.1 初始化数据目录

2.9.2 启动服务器

2.9.3 测试服务器

2.9.4 保护初始 MySQL 帐户

2.9.5 自动启动和停止 MySQL

2.10 Perl 安装注意事项

2.10.1 在 Unix 上安装 Perl

2.10.2 在 Windows 上安装 ActiveState Perl

2.10.3 使用 Perl DBI/DBD 接口时的问题

本章描述了如何获取和安装 MySQL。以下是程序概要，后续章节提供了详细信息。如果您计划将现有版本的 MySQL 升级到新版本而不是首次安装 MySQL，请参阅第三章，*升级 MySQL*，了解升级过程和升级前应考虑的问题。

如果您有兴趣从另一个数据库系统迁移到 MySQL，请参阅附录 A.8，“MySQL 8.0 FAQ：迁移”，其中包含有关迁移问题的一些常见问题的答案。

通常，MySQL 的安装遵循以下步骤：

1.  **确定 MySQL 是否在您的平台上运行并受支持。**

    请注意，并非所有平台都适合运行 MySQL，并且并非所有已知可运行 MySQL 的平台都得到 Oracle 公司的官方支持。有关官方支持的平台信息，请参阅 MySQL 网站上的[`www.mysql.com/support/supportedplatforms/database.html`](https://www.mysql.com/support/supportedplatforms/database.html)。

1.  **选择要安装的发行版。**

    MySQL 提供了多个版本，大多数版本都有几种分发格式可供选择。您可以选择包含二进制（预编译）程序或源代码的预打包分发。如果有疑问，请使用二进制分发。Oracle 还为那些想要查看最新开发并测试新代码的人提供了访问 MySQL 源代码的途径。要确定应使用哪个版本和分发类型，请参阅第 2.1.2 节，“应安装哪个 MySQL 版本和分发”。

1.  **选择要安装的跟踪。**

    MySQL 提供了一个修复错误跟踪（例如 MySQL 8.0），以及一个创新跟踪（今天是 MySQL 8.3），每个跟踪都针对不同的用例。这两个跟踪都被认为是可用于生产环境的，并包括错误修复，而创新发布还包括新功能和可能的修改行为。

    修复错误跟踪升级包括点版本，例如 MySQL 8.0.*`x`* 升级到 8.0.*`y`*，而创新跟踪发布通常只有次要版本，例如 MySQL 8.3.0 升级到 9.0.0。然而，创新跟踪确实偶尔有点版本发布。

1.  **下载要安装的分发。**

    有关说明，请参阅第 2.1.3 节，“如何获取 MySQL”。要验证分发的完整性，请使用第 2.1.4 节，“使用 MD5 校验和或 GnuPG 验证软件包完整性”中的说明。

1.  **安装分发。**

    要从二进制分发安装 MySQL，请使用第 2.2 节，“在 Unix/Linux 上使用通用二进制文件安装 MySQL”中的说明。或者，使用安全部署指南，该指南提供了部署 MySQL Enterprise Edition Server 通用二进制分发的程序，具有管理 MySQL 安装安全性功能的特性。

    要从源代码分发或当前开发源代码树安装 MySQL，请使用第 2.8 节，“从源代码安装 MySQL”中的说明。

1.  **执行任何必要的安装后设置。**

    安装 MySQL 后，请参阅第 2.9 节，“安装后设置和测试”，了解确保 MySQL 服务器正常工作的信息。还请参考第 2.9.4 节，“保护初始 MySQL 帐户”中提供的信息。本节描述了如何保护初始的 MySQL `root` 用户帐户，*该帐户在分配密码之前没有密码*。无论您是使用二进制还是源代码分发安装 MySQL，本节都适用。

1.  如果要运行 MySQL 基准测试脚本，则必须提供 Perl 支持。请参阅第 2.10 节，“Perl 安装说明”。

提供了在不同平台和环境上安装 MySQL 的说明，具体请参考各个平台：

+   **Unix，Linux**

    有关在大多数 Linux 和 Unix 平台上使用通用二进制文件（例如`.tar.gz`包）安装 MySQL 的说明，请参阅第 2.2 节，“在 Unix/Linux 上使用通用二进制文件安装 MySQL”。

    有关从源代码分发或源代码存储库构建 MySQL 的信息，请参阅第 2.8 节，“从源代码安装 MySQL”。

    有关在特定平台上安装、配置和从源代码构建的帮助，请参阅相应平台部分：

    +   Linux，包括特定发行版的安装方法，请参阅第 2.5 节，“在 Linux 上安装 MySQL”。

    +   IBM AIX，请参阅第 2.7 节，“在 Solaris 上安装 MySQL”。

+   **Microsoft Windows**

    有关在 Microsoft Windows 上安装 MySQL 的说明，可以使用 MySQL Installer 或 Zipped 二进制文件，请参阅第 2.3 节，“在 Microsoft Windows 上安装 MySQL”。

    有关使用 Microsoft Visual Studio 从源代码构建 MySQL 的详细说明，请参阅第 2.8 节，“从源代码安装 MySQL”。

+   **macOS**

    有关在 macOS 上安装 MySQL，包括使用二进制包和本机 PKG 格式，请参阅第 2.4 节，“在 macOS 上安装 MySQL”。

    有关如何利用 macOS Launch Daemon 自动启动和停止 MySQL 的信息，请参阅第 2.4.3 节，“安装和使用 MySQL Launch Daemon”。

    有关 MySQL Preference Pane 的信息，请参阅第 2.4.4 节，“安装和使用 MySQL Preference Pane”。
