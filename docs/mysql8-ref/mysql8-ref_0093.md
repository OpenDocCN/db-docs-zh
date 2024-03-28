# 2.9.2 启动服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/starting-server.html`](https://dev.mysql.com/doc/refman/8.0/en/starting-server.html)

2.9.2.1 启动 MySQL 服务器时出现问题的故障排除

本节描述了如何在 Unix 和类 Unix 系统上启动服务器。（对于 Windows，请参阅 Section 2.3.4.5, “首次启动服务器”。）有关一些建议的命令，您可以使用这些命令来测试服务器是否可访问并正常工作，请参阅 Section 2.9.3, “测试服务器”。

如果您的安装包含**mysqld_safe**，请像这样启动 MySQL 服务器：

```sql
$> bin/mysqld_safe --user=mysql &
```

注意

对于安装了 MySQL 的 Linux 系统，使用 RPM 包，服务器的启动和关闭是通过 systemd 管理的，而不是通过**mysqld_safe**，并且**mysqld_safe**未安装。请参阅 Section 2.5.9, “使用 systemd 管理 MySQL 服务器”。

如果您的安装包含 systemd 支持，请像这样启动服务器：

```sql
$> systemctl start mysqld
```

如果服务名称与 `mysqld` 不同，请替换为适当的服务名称（例如，在 SLES 系统上为 `mysql`）。

MySQL 服务器必须使用非特权（非`root`）登录帐户运行非常重要。为了确保这一点，请以 `root` 身份运行**mysqld_safe**，并包含如下所示的 `--user` 选项。否则，您应该以 `mysql` 登录后执行该程序，这种情况下，您可以在命令中省略 `--user` 选项。

有关以非特权用户身份运行 MySQL 的进一步说明，请参阅 Section 8.1.5, “如何以普通用户身份运行 MySQL”。

如果命令立即失败并打印 `mysqld ended`，请查看错误日志中的信息（默认情况下是数据��录中的 `*`host_name`*.err` 文件）。

如果服务器无法访问数据目录，启动或读取 `mysql` 模式中的授权表，它会在错误日志中写入一条消息。如果您在继续此步骤之前忽略了通过初始化数据目录创建授权表，或者如果您在运行初始化数据目录的命令时没有使用 `--user` 选项，可能会出现此类问题。删除 `data` 目录并使用 `--user` 选项运行该命令。

如果您在启动服务器时遇到其他问题，请参见第 2.9.2.1 节，“启动 MySQL 服务器时出现问题的故障排除”。有关**mysqld_safe**的更多信息，请参见第 6.3.2 节，“mysqld_safe — MySQL 服务器启动脚本”。有关 systemd 支持的更多信息，请参见第 2.5.9 节，“使用 systemd 管理 MySQL 服务器”。
