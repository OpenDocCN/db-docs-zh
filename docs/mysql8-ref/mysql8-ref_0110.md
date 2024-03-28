# 3.8 使用 MySQL Yum 存储库升级 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/updating-yum-repo.html`](https://dev.mysql.com/doc/refman/8.0/en/updating-yum-repo.html)

对于支持 Yum 的平台（参见第 2.5.1 节，“使用 MySQL Yum 存储库在 Linux 上安装 MySQL”，有一个列表），您可以使用 MySQL Yum 存储库执行就地升级 MySQL（即，替换旧版本，然后使用旧数据文件运行新版本）。

注意事项

+   在对 MySQL 执行任何更新之前，请仔细遵循第三章，*升级 MySQL*中的说明。在那里讨论的其他说明中，特别重要的是在更新之前备份您的数据库。

+   以下说明假设您已经使用 MySQL Yum 存储库或直接从[MySQL 开发者区的 MySQL 下载页面](https://dev.mysql.com/downloads/)下载的 RPM 包安装了 MySQL；如果不是这种情况，请按照使用 MySQL Yum 存储库替换 MySQL 的第三方发行版中的说明操作。

1.  ### 选择目标系列

    默认情况下，MySQL Yum 存储库将 MySQL 更新到您在安装过程中选择的发布系列的最新版本（有关详细信息，请参见选择发布系列），这意味着，例如，5.7.x 安装 *不会* 自动更新到 8.0.x 版本。要更新到另一个发布系列，您必须首先禁用已选择的系列的子存储库（默认情况下或自行选择），然后启用目标系列的子存储库。要执行此操作，请参见选择发布系列中给出的一般说明。对于从 MySQL 5.7 升级到 8.0，请执行选择发布系列中所示步骤的*相反*操作，禁用 MySQL 5.7 系列的子存储库，并启用 MySQL 8.0 系列的子存储库。

    一般规则是，要从一个发布系列升级到另一个发布系列，请先转到下一个系列，而不是跳过一个系列。例如，如果您当前运行 MySQL 5.6 并希望升级到 8.0，请先升级到 MySQL 5.7，然后再升级到 8.0。

    重要

    有关从 MySQL 5.7 升级到 8.0 的重要信息，请参阅从 MySQL 5.7 升级到 8.0。

1.  ### 升级 MySQL

    通过以下命令升级 MySQL 及其组件，对于不支持 dnf 的平台：

    ```sql
    sudo yum update mysql-server
    ```

    对于支持 dnf 的平台：

    ```sql
    sudo dnf upgrade mysql-server
    ```

    或者，您可以通过告诉 Yum 更新系统上的所有内容来更新 MySQL，这可能需要更多时间。对于未启用 dnf 的平台：

    ```sql
    sudo yum update
    ```

    对于启用 dnf 的平台：

    ```sql
    sudo dnf upgrade
    ```

1.  ### 重新启动 MySQL

    MySQL 服务器在通过 Yum 更新后总是重新启动。在 MySQL 8.0.16 之前，服务器重新启动后运行**mysql_upgrade**来检查并可能解决旧数据与升级软件之间的任何不兼容性。**mysql_upgrade**还执行其他功能；有关详细信息，请参见 Section 6.4.5，“mysql_upgrade — 检查和升级 MySQL 表”。从 MySQL 8.0.16 开始，不再需要此步骤，因为服务器执行了以前由**mysql_upgrade**处理的所有任务。

您还可以仅更新特定组件。使用以下命令列出所有已安装的 MySQL 组件的软件包（对于启用 dnf 的系统，请将命令中的 **yum** 替换为 **dnf**）：

```sql
sudo yum list installed | grep "^mysql"
```

在确定您选择的组件的软件包名称后，使用以下命令更新软件包，将 *`package-name`* 替换为软件包的名称。对于未启用 dnf 的平台：

```sql
sudo yum update *package-name*
```

对于启用 dnf 的平台：

```sql
sudo dnf upgrade *package-name*
```

### 升级共享客户端库

使用 Yum 仓库更新 MySQL 后，使用旧版本共享客户端库编译的应用程序应继续工作。

*如果重新编译应用程序并动态链接它们与更新的库：*与新版本共享库一样，新旧库之间的符号版本之间存在差异或添加（例如，在新的标准 8.0 共享客户端库和一些较旧的—之前或变体—版本之间存在符号版本之间的差异或添加，这些共享库是由 Linux 发行版的软件仓库原生提供的，或来自其他来源），在这些更新的新共享库上编译的任何应用程序需要在部署应用程序的系统上使用这些更新的库。如预期的那样，如果这些库不在位，需要共享库的应用程序将失败。因此，请确保在这些系统上部署来自 MySQL 的共享库软件包。为此，请将 MySQL Yum 仓库添加到系统中（请参见添加 MySQL Yum 仓库）并按照使用 Yum 安装其他 MySQL 产品和组件中给出的说明安装最新的共享库。
