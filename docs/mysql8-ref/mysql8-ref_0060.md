# 2.4.1 在 macOS 上安装 MySQL 的一般注意事项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/macos-installation-notes.html`](https://dev.mysql.com/doc/refman/8.0/en/macos-installation-notes.html)

您应该牢记以下问题和注意事项：

+   **其他 MySQL 安装**：安装过程不识别 Homebrew 等软件包管理器安装的 MySQL。安装和升级过程适用于我们提供的 MySQL 软件包。如果存在其他安装，请考虑在执行此安装程序之前停止它们，以避免端口冲突。

    **Homebrew**：例如，如果您使用 Homebrew 将 MySQL Server 安装到默认位置，则 MySQL 安装程序将安装到不同位置，并且不会升级 Homebrew 中的版本。在这种情况下，您将拥有多个 MySQL 安装，这些安装默认情况下会尝试使用相同的端口。在运行此安装程序之前，请停止其他 MySQL Server 实例，例如执行*brew services stop mysql*来停止 Homebrew 的 MySQL 服务。

+   **Launchd**：安装了一个 launchd 守护程序，用于更改 MySQL 配置选项。如有需要，请考虑编辑它，有关更多信息，请参阅下面的文档。此外，macOS 10.10 删除了启动项支持，改为使用 launchd 守护程序。macOS 系统偏好设置下的可选 MySQL 首选项窗格使用 launchd 守护程序。

+   **用户**：您可能需要（或想要）创建一个特定的`mysql`用户来拥有 MySQL 目录和数据。您可以通过**Directory Utility**来完成这个操作，`mysql`用户应该已经存在。在单用户模式下使用时，系统`/etc/passwd`文件中应该已经存在一个`_mysql`（注意下划线前缀）的条目。

+   **数据**：由于 MySQL 软件包安装程序将 MySQL 内容安装到特定版本和平台的目录中，您可以使用此功能在不同版本之间升级和迁移数据库。您需要将旧版本的`data`目录复制到新版本，或者指定一个替代的`datadir`值来设置数据目录的位置。默认情况下，MySQL 目录安装在`/usr/local/`下。

+   **别名**：您可能希望将别名添加到您的 shell 资源文件中，以便更轻松地访问常用程序，例如**mysql**和**mysqladmin**。在**bash**中的语法是：

    ```sql
    alias mysql=/usr/local/mysql/bin/mysql
    alias mysqladmin=/usr/local/mysql/bin/mysqladmin
    ```

    对于**tcsh**，使用：

    ```sql
    alias mysql /usr/local/mysql/bin/mysql
    alias mysqladmin /usr/local/mysql/bin/mysqladmin
    ```

    更好的做法是将`/usr/local/mysql/bin`添加到您的`PATH`环境变量中。您可以通过修改适合您的 shell 的相应启动文件来实现这一点。有关更多信息，请参阅 Section 6.2.1, “Invoking MySQL Programs”。

+   **移除**：在您已经从先前安装中复制了 MySQL 数据库文件并成功启动了新服务器之后，您应该考虑移除旧安装文件以节省磁盘空间。此外，您还应该移除位于`/Library/Receipts/mysql-*`VERSION`*.pkg`目录中的旧版本软件包安装目录。
