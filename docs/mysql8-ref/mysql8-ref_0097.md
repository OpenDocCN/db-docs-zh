# 2.9.5 自动启动和停止 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/automatic-start.html`](https://dev.mysql.com/doc/refman/8.0/en/automatic-start.html)

本节讨论了启动和停止 MySQL 服务器的方法。

通常，您可以通过以下方式之一启动**mysqld**服务器：

+   直接调用**mysqld**。这在任何平台上都适用。

+   在 Windows 上，您可以设置一个 MySQL 服务，该服务在 Windows 启动时自动运行。请参阅 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

+   在 Unix 和类 Unix 系统上，您可以调用**mysqld_safe**，它会尝试确定**mysqld**的正确选项，然后使用这些选项运行它。请参阅 Section 6.3.2, “mysqld_safe — MySQL Server Startup Script”。

+   在支持 systemd 的 Linux 系统上，您可以使用它来控制服务器。请参阅 Section 2.5.9, “Managing MySQL Server with systemd”。

+   在使用 System V 风格运行目录（即 `/etc/init.d` 和特定运行级别目录）的系统上，调用**mysql.server**。该脚本主要用于系统启动和关闭。它通常以 `mysql` 的名称安装。**mysql.server**脚本通过调用**mysqld_safe**来启动服务器。请参阅 Section 6.3.3, “mysql.server — MySQL Server Startup Script”。

+   在 macOS 上，安装一个 launchd 守护程序以在系统启动时启用自动 MySQL 启动。该守护程序通过调用**mysqld_safe**来启动服务器。详情请参阅 Section 2.4.3, “Installing and Using the MySQL Launch Daemon”。MySQL 首选项面板还提供了通过系统偏好设置启动和停止 MySQL 的控制。请参阅 Section 2.4.4, “Installing and Using the MySQL Preference Pane”。

+   在 Solaris 上，使用服务管理框架（SMF）系统来启动和控制 MySQL 启动。

systemd，**mysqld_safe** 和 **mysql.server** 脚本，Solaris SMF，以及 macOS 启动项（或 MySQL 首选项面板）可用于手动启动服务器，或在系统启动时自动启动。systemd，**mysql.server** 和启动项也可用于停止服务器。

以下表显示服务器和启动脚本从选项文件中读取的选项组。

**表 2.15 MySQL 启动脚本和支持的服务器选项组**

| 脚本 | 选项组 |
| --- | --- |
| **mysqld** | `[mysqld]`, `[server]`, `[mysqld-*`major_version`*]` |
| **mysqld_safe** | `[mysqld]`, `[server]`, `[mysqld_safe]` |
| **mysql.server** | `[mysqld]`, `[mysql.server]`, `[server]` |

`[mysqld-*`major_version`*]` 意味着像 `[mysqld-5.7]` 和 `[mysqld-8.0]` 这样的名称组将被版本为 5.7.x、8.0.x 等的服务器读取。此功能可用于指定仅可由给定发布系列内的服务器读取的选项。

为了向后兼容，**mysql.server** 还会读取 `[mysql_server]` 组，而 **mysqld_safe** 也会读取 `[safe_mysqld]` 组。为了保持最新，您应该更新您的选项文件以使用 `[mysql.server]` 和 `[mysqld_safe]` 组。 

有关 MySQL 配置文件及其结构和内容的更多信息，请参见 Section 6.2.2.2, “使用选项文件”。
