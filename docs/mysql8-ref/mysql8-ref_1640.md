> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-binary.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-binary.html)

#### 25.3.1.1 在 Linux 上安装 NDB 集群二进制发行版

本节涵盖了从 Oracle 提供的预编译二进制文件中为每种类型的集群节点安装正确可执行文件所需的步骤。

对于使用预编译二进制文件设置集群，每个集群主机安装过程的第一步是从 [NDB 集群下载页面](https://dev.mysql.com/downloads/cluster/) 下载二进制存档。 （对于最新的 64 位 NDB 8.0 版本，这是 `mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64.tar.gz`。）我们假设您已将此文件放置在每台机器的 `/var/tmp` 目录中。

如果需要自定义二进制文件，请参见 第 2.8.5 节，“使用开发源码树安装 MySQL”。

注意

完成安装后，不要立即启动任何二进制文件。我们将在配置节点后向您展示如何操作（参见 第 25.3.3 节，“NDB 集群的初始配置”）。

**SQL 节点。** 在每台指定为托管 SQL 节点的机器上，以系统 `root` 用户身份执行以下步骤：

1.  检查您的 `/etc/passwd` 和 `/etc/group` 文件（或使用操作系统提供的用于管理用户和组的工具），查看系统上是否已经存在 `mysql` 组和 `mysql` 用户。 一些操作系统发行版在操作系统安装过程中会创建这些。 如果它们尚不存在，请创建一个新的 `mysql` 用户组，然后将 `mysql` 用户添加到此组中：

    ```sql
    $> groupadd mysql
    $> useradd -g mysql -s /bin/false mysql
    ```

    **useradd** 和 **groupadd** 的语法在不同版本的 Unix 上可能略有不同，或者可能有不同的名称，如 **adduser** 和 **addgroup**。

1.  切换到包含下载文件的目录，解压缩存档，并创建一个名为 `mysql` 的符号链接指向 `mysql` 目录。

    注意

    实际文件和目录名称根据 NDB 集群版本号而异。

    ```sql
    $> cd /var/tmp
    $> tar -C /usr/local -xzvf mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64.tar.gz
    $> ln -s /usr/local/mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64 /usr/local/mysql
    ```

1.  切换到 `mysql` 目录，并使用 **mysqld** `--initialize` 来设置系统数据库，如下所示：

    ```sql
    $> cd mysql
    $> mysqld --initialize
    ```

    这为 MySQL `root`帐户生成一个随机密码。如果您*不*希望生成随机密码，可以将`--initialize-insecure`选项替换为`--initialize`。无论哪种情况，您都应在执行此步骤之前查看 Section 2.9.1, “Initializing the Data Directory”，以获取更多信息。另请参阅 Section 6.4.2, “mysql_secure_installation — Improve MySQL Installation Security”。

1.  为 MySQL 服务器和数据目录设置必要的权限：

    ```sql
    $> chown -R root .
    $> chown -R mysql data
    $> chgrp -R mysql .
    ```

1.  将 MySQL 启动脚本复制到适当的目录，使其可执行，并设置在操作系统启动时启动：

    ```sql
    $> cp support-files/mysql.server /etc/rc.d/init.d/
    $> chmod +x /etc/rc.d/init.d/mysql.server
    $> chkconfig --add mysql.server
    ```

    （启动脚本目录可能因操作系统和版本而异，例如，在某些 Linux 发行版中，它是`/etc/init.d`。）

    在这里，我们使用 Red Hat 的**chkconfig**来创建到启动脚本的链接；在您的平台上使用适当的方式，如 Debian 上的**update-rc.d**。

请记住，上述步骤必须在每台要放置 SQL 节点的机器上重复执行。

**数据节点。** 数据节点的安装不需要**mysqld**二进制文件。只需要 NDB Cluster 数据节点可执行文件**ndbd**（单线程）或**ndbmtd**")（多线程）。这些二进制文件也可以在`.tar.gz`存档中找到。同样，我们假设您已将此存档放置在`/var/tmp`中。

作为系统`root`（即在使用**sudo**、**su root**或您系统的等效方式暂时假���系统管理员帐户特权之后），执行以下步骤在数据节点主机上安装数据节点二进制文件：

1.  将位置更改为`/var/tmp`目录，并从存档中提取**ndbd**和**ndbmtd**")二进制文件到适当的目录，如`/usr/local/bin`：

    ```sql
    $> cd /var/tmp
    $> tar -zxvf mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64.tar.gz
    $> cd mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64
    $> cp bin/ndbd /usr/local/bin/ndbd
    $> cp bin/ndbmtd /usr/local/bin/ndbmtd
    ```

    （一旦**ndb_mgm**和**ndb_mgmd**已复制到可执行文件目录中，您可以安全地删除从`/var/tmp`解压下载的存档创建的目录及其包含的文件。）

1.  将位置更改为您复制文件的目录，然后使这两个文件都可执行：

    ```sql
    $> cd /usr/local/bin
    $> chmod +x ndb*
    ```

上述步骤应在每个数据节点主机上重复执行。

虽然只需要运行一个数据节点可执行文件来运行 NDB 集群数据节点，但我们在前面的说明中已经向您展示了如何安装**ndbd**和**ndbmtd**。我们建议您在安装或升级 NDB 集群时执行此操作，即使您打算只使用其中一个，因为这样可以节省时间和麻烦，以防以后决定从一个切换到另一个。

注意

每台托管数据节点的机器上的数据目录是`/usr/local/mysql/data`。在配置管理节点时，这些信息至关重要。（参见第 25.3.3 节，“NDB 集群的初始配置”。）

**管理节点。** 安装管理节点不需要**mysqld**二进制文件。只需要 NDB 集群管理服务器(**ndb_mgmd**)；您很可能也想安装管理客户端(**ndb_mgm**)。这两个二进制文件也可以在`.tar.gz`存档文件中找到。同样，我们假设您已将此存档文件放在`/var/tmp`中。

作为系统`root`，执行以下步骤在管理节点主机上安装**ndb_mgmd**和**ndb_mgm**：

1.  切换到`/var/tmp`目录，并将**ndb_mgm**和**ndb_mgmd**从存档中提取到一个合适的目录，如`/usr/local/bin`：

    ```sql
    $> cd /var/tmp
    $> tar -zxvf mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64.tar.gz
    $> cd mysql-cluster-gpl-8.0.34-linux-glibc2.12-x86_64
    $> cp bin/ndb_mgm* /usr/local/bin
    ```

    (一旦**ndb_mgm**和**ndb_mgmd**已被复制到可执行文件目录中，您可以安全地删除从`/var/tmp`解压下载存档文件时创建的目录及其包含的文件。)

1.  切换到您复制文件的目录，然后使这两个文件都可执行：

    ```sql
    $> cd /usr/local/bin
    $> chmod +x ndb_mgm*
    ```

在第 25.3.3 节，“NDB 集群的初始配置”中，我们为示例 NDB 集群中的所有节点创建配置文件。
