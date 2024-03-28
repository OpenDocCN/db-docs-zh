# 2.5.7 在 Linux 上从本地软件仓库安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/linux-installation-native.html`](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-native.html)

许多 Linux 发行版在其本地软件仓库中包含 MySQL 服务器、客户端工具和开发组件的版本，并可以使用平台的标准软件包管理系统进行安装。本节提供了使用这些软件包管理系统安装 MySQL 的基本说明。

重要

本地软件包通常落后于当前可用版本。通常无法安装开发里程碑版本（DMR），因为这些版本通常不会在本地仓库中提供。在继续之前，我们建议您查看 Section 2.5, “在 Linux 上安装 MySQL”中描述的其他安装选项。

下面显示了特定于发行版的说明：

+   **Red Hat Linux, Fedora, CentOS**

    注意

    对于许多 Linux 发行版，您可以使用 MySQL Yum 仓库而不是平台的本地软件仓库来安装 MySQL。有关详细信息，请参阅 Section 2.5.1, “使用 MySQL Yum 仓库在 Linux 上安装 MySQL”。

    对于 Red Hat 和类似的发行版，MySQL 分发为多个单独的软件包，`mysql` 用于客户端工具，`mysql-server` 用于服务器和相关工具，`mysql-libs` 用于库。如果您希望从不同的语言和环境（如 Perl、Python 等）提供连接，则需要这些库。

    要安装，请使用**yum**命令指定要安装的软件包。例如：

    ```sql
    #> yum install mysql mysql-server mysql-libs mysql-server
    Loaded plugins: presto, refresh-packagekit
    Setting up Install Process
    Resolving Dependencies
    --> Running transaction check
    ---> Package mysql.x86_64 0:5.1.48-2.fc13 set to be updated
    ---> Package mysql-libs.x86_64 0:5.1.48-2.fc13 set to be updated
    ---> Package mysql-server.x86_64 0:5.1.48-2.fc13 set to be updated
    --> Processing Dependency: perl-DBD-MySQL for package: mysql-server-5.1.48-2.fc13.x86_64
    --> Running transaction check
    ---> Package perl-DBD-MySQL.x86_64 0:4.017-1.fc13 set to be updated
    --> Finished Dependency Resolution

    Dependencies Resolved

    ================================================================================
     Package               Arch          Version               Repository      Size
    ================================================================================
    Installing:
     mysql                 x86_64        5.1.48-2.fc13         updates        889 k
     mysql-libs            x86_64        5.1.48-2.fc13         updates        1.2 M
     mysql-server          x86_64        5.1.48-2.fc13         updates        8.1 M
    Installing for dependencies:
     perl-DBD-MySQL        x86_64        4.017-1.fc13          updates        136 k

    Transaction Summary
    ================================================================================
    Install       4 Package(s)
    Upgrade       0 Package(s)

    Total download size: 10 M
    Installed size: 30 M
    Is this ok [y/N]: y
    Downloading Packages:
    Setting up and reading Presto delta metadata
    Processing delta metadata
    Package(s) data still to download: 10 M
    (1/4): mysql-5.1.48-2.fc13.x86_64.rpm                    | 889 kB     00:04
    (2/4): mysql-libs-5.1.48-2.fc13.x86_64.rpm               | 1.2 MB     00:06
    (3/4): mysql-server-5.1.48-2.fc13.x86_64.rpm             | 8.1 MB     00:40
    (4/4): perl-DBD-MySQL-4.017-1.fc13.x86_64.rpm            | 136 kB     00:00
    --------------------------------------------------------------------------------
    Total                                           201 kB/s |  10 MB     00:52
    Running rpm_check_debug
    Running Transaction Test
    Transaction Test Succeeded
    Running Transaction
      Installing     : mysql-libs-5.1.48-2.fc13.x86_64                          1/4
      Installing     : mysql-5.1.48-2.fc13.x86_64                               2/4
      Installing     : perl-DBD-MySQL-4.017-1.fc13.x86_64                       3/4
      Installing     : mysql-server-5.1.48-2.fc13.x86_64                        4/4

    Installed:
      mysql.x86_64 0:5.1.48-2.fc13            mysql-libs.x86_64 0:5.1.48-2.fc13
      mysql-server.x86_64 0:5.1.48-2.fc13

    Dependency Installed:
      perl-DBD-MySQL.x86_64 0:4.017-1.fc13

    Complete!
    ```

    MySQL 和 MySQL 服务器现在应该已安装。一个示例配置文件被安装到`/etc/my.cnf`。要启动 MySQL 服务器，请使用**systemctl**：

    ```sql
    $> systemctl start mysqld
    ```

    如果数据库表尚不存在，系统会自动为您创建这些表。但是，您应该运行**mysql_secure_installation**来设置服务器的 root 密码。

+   **Debian, Ubuntu, Kubuntu**

    注意

    对于支持的 Debian 和 Ubuntu 版本，可以使用[MySQL APT 仓库](https://dev.mysql.com/downloads/repo/apt/)来安装 MySQL，而不是使用平台的本地软件仓库。有关详细信息，请参阅 Section 2.5.2, “使用 MySQL APT 仓库在 Linux 上安装 MySQL”。

    在 Debian 及相关发行版中，他们的软件仓库中有两个 MySQL 软件包，`mysql-client`和`mysql-server`，分别用于客户端和服务器组件。您应该指定一个明确的版本，例如`mysql-client-5.1`，以确保安装您想要的 MySQL 版本。

    要下载和安装，包括任何依赖项，请使用**apt-get**命令，指定您想要安装的软件包。

    注意

    在安装之前，请确保更新您的`apt-get`索引文件，以确保您下载的是最新版本。

    注意

    **apt-get**命令安装了许多软件包，包括 MySQL 服务器，以提供典型的工具和应用环境。这意味着除了主要的 MySQL 软件包外，您可能还会安装大量软件包。

    在安装过程中，会创建初始数据库，并提示您输入 MySQL root 密码（以及确认密码）。在`/etc/mysql/my.cnf`中创建配置文件。在`/etc/init.d/mysql`中创建初始化脚本。

    服务器应该已经启动。您可以使用以下命令手动启动和停止服务器：

    ```sql
    #> service mysql [start|stop]
    ```

    该服务会自动添加到 2、3 和 4 运行级别，并在单个、关机和重启级别中添加停止脚本。
