> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-extract-archive.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-extract-archive.html)

#### 2.3.4.1 提取安装存档

要手动安装 MySQL，请执行以下操作：

1.  如果您正在从以前的版本升级，请在开始升级过程之前参考 Section 3.11, “Upgrading MySQL on Windows”。

1.  确保您以具有管理员权限的用户登录。

1.  选择安装位置。传统上，MySQL 服务器安装在`C:\mysql`中。如果您没有在`C:\mysql`安装 MySQL，则必须在启动时或在选项文件中指定安装目录的路径。请参阅 Section 2.3.4.2, “Creating an Option File”。

    注意

    MySQL 安装程序将 MySQL 安装在`C:\Program Files\MySQL`下。

1.  使用您喜欢的文件压缩工具将安装存档提取到选择的安装位置。一些工具可能会将存档提取到所选安装位置内的一个文件夹中。如果发生这种情况，您可以将子文件夹的内容移动到所选的安装位置。
