# 2.5.5 使用 Oracle 的 Debian 软件包在 Linux 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/linux-installation-debian.html`](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-debian.html)

Oracle 提供了用于在 Debian 或类似 Debian 的 Linux 系统上安装 MySQL 的 Debian 软件包。这些软件包通过两个不同的渠道提供：

+   [MySQL APT 存储库](https://dev.mysql.com/downloads/repo/apt/)。这是在类似 Debian 的系统上安装 MySQL 的首选方法，因为它提供了一种简单方便的方式来安装和更新 MySQL 产品。详情请参阅 Section 2.5.2, “使用 MySQL APT 存储库在 Linux 上安装 MySQL”。

+   [MySQL 开发者区下载区](https://dev.mysql.com/downloads/)。详情请参阅 Section 2.1.3, “如何获取 MySQL”。以下是有关那里提供的 Debian 软件包以及安装说明的一些信息：

    +   MySQL 开发者区提供了各种 Debian 软件包，用于在当前的 Debian 和 Ubuntu 平台上安装 MySQL 的不同组件。首选方法是使用 tarball 捆绑包，其中包含了 MySQL 基本设置所需的软件包。tarball 捆绑包的名称格式为`mysql-server_*`MVER`*-*`DVER`*_*`CPU`*.deb-bundle.tar`。*`MVER`*是 MySQL 版本，*`DVER`*是 Linux 发行版版本。*`CPU`*值表示软件包构建的处理器类型或系列，如下表所示：

        **表 2.13 MySQL Debian 和 Ubuntu 安装软件包 CPU 标识符**

        | *`CPU`* 值 | 预期处理器类型或系列 |
        | --- | --- |
        | `i386` | 奔腾处理器或更高版本，32 位 |
        | `amd64` | 64 位 x86 处理器 |

    +   下载 tarball 后，使用以下命令解压缩：

        ```sql
        $> tar -xvf mysql-server_*MVER*-*DVER*_*CPU*.deb-bundle.tar
        ```

    +   如果系统中尚未安装`libaio`库，则可能需要安装：

        ```sql
        $> sudo apt-get install libaio1
        ```

    +   使用以下命令预配置 MySQL 服务器软件包：

        ```sql
        $> sudo dpkg-preconfigure mysql-community-server_*.deb
        ```

        您将被要求为 MySQL 安装的 root 用户提供密码。您可能还会被问及有关安装的其他问题。

        重要

        确保记住您设置的 root 密码。想要稍后设置密码的用户可以在对话框中将密码字段留空，然后只需按下 OK；在这种情况下，使用 Unix 套接字文件进行连接的服务器的 root 访问权限是通过 MySQL 套接字对等凭证认证插件进行验证。您可以稍后使用**mysql_secure_installation**设置 root 密码。

    +   对于 MySQL 服务器的基本安装，安装数据库公共文件包、客户端包、客户端元包、服务器包和服务器元包（按照这个顺序）；你可以使用一个命令完成这些操作：

        ```sql
        $> sudo dpkg -i mysql-{common,community-client-plugins,community-client-core,community-client,client,community-server-core,community-server,server}_*.deb
        ```

        还有一些包的包名中带有`server-core`和`client-core`。这些只包含二进制文件，并且会被标准包自动安装。单独安装它们并不会导致 MySQL 的正常运行。

        如果**dpkg**（如 libmecab2）提示存在未满足的依赖关系，你可以使用**apt-get**来解决：

        ```sql
        sudo apt-get -f install
        ```

        下面是系统上文件的安装位置：

        +   所有配置文件（如`my.cnf`）都位于`/etc/mysql`

        +   所有二进制文件、库文件、头文件等都位于`/usr/bin`和`/usr/sbin`

        +   数据目录位于`/var/lib/mysql`

注意

Debian 发行版的 MySQL 也由其他供应商提供。请注意，它们可能在功能、能力和约定（包括通信设置）方面与 Oracle 构建的版本不同，并且本手册中的说明不一定适用于安装它们。应该参考供应商的说明。
