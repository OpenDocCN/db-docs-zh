# 2.3.1 Microsoft Windows 上的 MySQL 安装布局

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-installation-layout.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-installation-layout.html)

对于 Windows 上的 MySQL 8.0，默认安装目录是 `C:\Program Files\MySQL\MySQL Server 8.0`，如果使用 MySQL Installer 进行安装。如果您使用 ZIP 存档方法安装 MySQL，则可能更喜欢安装在 `C:\mysql`。但是，子目录的布局保持不变。

所有文件都位于此父目录中，使用下表所示的结构。

**表 2.4 Microsoft Windows 上默认的 MySQL 安装布局**

| 目录 | 目录内容 | 备注 |
| --- | --- | --- |
| `bin` | **mysqld** 服务器，客户端和实用程序 |  |
| `%PROGRAMDATA%\MySQL\MySQL Server 8.0\` | 日志文件，数据库 | Windows 系统变量 `%PROGRAMDATA%` 默认为 `C:\ProgramData`。 |
| `docs` | 发行文档 | 使用 MySQL Installer，使用 `Modify` 操作选择此可选文件夹。 |
| `include` | 包含（头）文件 |  |
| `lib` | 库文件 |  |
| `share` | 杂项支持文件，包括错误消息，字符集文件，示例配置文件，用于数据库安装的 SQL |  |
