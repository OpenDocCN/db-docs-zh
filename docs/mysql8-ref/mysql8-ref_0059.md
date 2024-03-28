# 2.4 在 macOS 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/macos-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/macos-installation.html)

2.4.1 在 macOS 上安装 MySQL 的一般注意事项

2.4.2 使用本机软件包在 macOS 上安装 MySQL

2.4.3 安装和使用 MySQL 启动守护程序

2.4.4 安装和使用 MySQL 首选项窗格

有关 MySQL 服务器支持的 macOS 版本列表，请参阅 [`www.mysql.com/support/supportedplatforms/database.html`](https://www.mysql.com/support/supportedplatforms/database.html)。

macOS 上的 MySQL 可以以多种不同形式使用：

+   本机软件包安装程序，使用本机 macOS 安装程序（DMG）引导您完成 MySQL 的安装过程。有关更多信息，请参阅 第 2.4.2 节，“使用本机软件包在 macOS 上安装 MySQL”。您可以在 macOS 上使用软件包安装程序。执行安装的用户必须具有管理员权限。

+   压缩的 TAR 存档，使用 Unix 的 **tar** 和 **gzip** 命令打包的文件。要使用这种方法，您需要打开一个 **终端** 窗口。使用此方法不需要管理员权限；您可以在任何地方安装 MySQL 服务器。有关使用此方法的更多信息，您可以使用通用的 tarball 使用说明，第 2.2 节，“在 Unix/Linux 上使用通用二进制文件安装 MySQL”。

    除了核心安装外，软件包安装程序还包括 第 2.4.3 节，“安装和使用 MySQL 启动守护程序” 和 第 2.4.4 节，“安装和使用 MySQL 首选项窗格”，以简化您的安装管理。

有关在 macOS 上使用 MySQL 的其他信息，请参阅 第 2.4.1 节，“在 macOS 上安装 MySQL 的一般注意事项”。
