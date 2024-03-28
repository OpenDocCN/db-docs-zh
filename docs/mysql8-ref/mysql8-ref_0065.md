# 2.5.1 使用 MySQL Yum 存储库在 Linux 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html`](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html)

适用于 Oracle Linux、Red Hat Enterprise Linux、CentOS 和 Fedora 的[MySQL Yum 存储库](https://dev.mysql.com/downloads/repo/yum/)提供了用于安装 MySQL 服务器、客户端、MySQL Workbench、MySQL Utilities、MySQL Router、MySQL Shell、Connector/ODBC、Connector/Python 等的 RPM 软件包（并非所有软件包都适用于所有发行版；有关详细信息，请参阅使用 Yum 安装其他 MySQL 产品和组件）。

#### 在开始之前

作为一款流行的开源软件，MySQL 以其原始或重新打包的形式广泛安装在许多系统上，这些系统来自不同的来源，包括不同的软件下载站点、软件存储库等。以下说明假定 MySQL 尚未使用第三方分发的 RPM 软件包安装在您的系统上；如果不是这种情况，请按照第 3.8 节，“使用 MySQL Yum 存储库升级 MySQL”或使用 MySQL Yum 存储库替换第三方分发的 MySQL 中给出的说明进行操作。

注意

存储库设置的 RPM 文件名以`mysql80-community`开头，以突出显示默认的活动 MySQL 子存储库，即今天的 MySQL 8.0。它们还安装了用于 MySQL 8.3 创新版本安装和升级的 MySQL 创新跟踪子存储库。

#### 安装 MySQL 的新安装步骤

按照以下步骤使用 MySQL Yum 存储库安装最新的 GA 版本 MySQL：

1.  #### 添加 MySQL Yum 存储库

    首先，将 MySQL Yum 存储库添加到系统的存储库列表中。这是一个一次性操作，可以通过安装 MySQL 提供的一个 RPM 来执行。按照以下步骤进行：

    1.  前往 MySQL 开发者区的下载 MySQL Yum 存储库页面（[`dev.mysql.com/downloads/repo/yum/`](https://dev.mysql.com/downloads/repo/yum/)）。

    1.  选择并下载适用于您平台的发布软件包。

    1.  使用以下命令安装下载的发布软件包，将*`platform-and-version-specific-package-name`*替换为下载的 RPM 软件包的名称：

        ```sql
        $> sudo yum install *platform-and-version-specific-package-name*.rpm
        ```

        对于基于 EL6 的系统，命令的形式为：

        ```sql
        $> sudo yum install mysql80-community-release-el6-*{version-number}*.noarch.rpm
        ```

        对于基于 EL7 的系统：

        ```sql
        $> sudo yum install mysql80-community-release-el7-*{version-number}*.noarch.rpm
        ```

        对于基于 EL8 的系统：

        ```sql
        $> sudo yum install mysql80-community-release-el8-*{version-number}*.noarch.rpm
        ```

        对于基于 EL9 的系统：

        ```sql
        $> sudo yum install mysql80-community-release-el9-*{version-number}*.noarch.rpm
        ```

        对于 Fedora 37：

        ```sql
        $> sudo dnf install mysql80-community-release-fc37-*{version-number}*.noarch.rpm
        ```

        对于 Fedora 38：

        ```sql
        $> sudo dnf install mysql80-community-release-fc38-*{version-number}*.noarch.rpm
        ```

        对于 Fedora 39：

        ```sql
        $> sudo dnf install mysql80-community-release-fc39-*{version-number}*.noarch.rpm
        ```

        安装命令将 MySQL Yum 存储库添加到系统的存储库列表中，并下载 GnuPG 密钥以检查软件包的完整性。有关使用 GnuPG 密钥检查的详细信息，请参阅第 2.1.4.2 节，“使用 GnuPG 进行签名检查”。

        您可以通过以下命令检查 MySQL Yum 存储库是否已成功添加（对于启用了 dnf 的系统，请在命令中用**dnf**替换**yum**）：

        ```sql
        $> yum repolist enabled | grep "mysql.*-community.*"
        ```

    注意

    一旦在您的系统上启用了 MySQL Yum 存储库，通过**yum update**命令（或对于启用了 dnf 的系统，通过**dnf upgrade**）进行的任何系统范围的更新都会升级系统上的 MySQL 软件包，并在 MySQL Yum 存储库中找到替代品时替换任何本地第三方软件包；请参见第 3.8 节，“使用 MySQL Yum 存储库升级 MySQL”，讨论可能对系统产生的一些影响，请参见升级共享客户端库。

1.  #### 选择一个发布系列

    当使用 MySQL Yum 存储库时，默认选择安装最新的 GA 系列（目前为 MySQL 8.0）。如果这正是您想要的，请跳转到下一步，安装 MySQL。

    在 MySQL Yum 存储库中，MySQL Community Server 的不同发布系列托管在不同的子存储库中。默认情况下启用最新 GA 系列（目前为 MySQL 8.0）的子存储库，并默认禁用所有其他系列的子存储库（例如 MySQL 8.0 系列）。使用此命令查看 MySQL Yum 存储库中的所有子存储库，并查看它们中哪些是启用的或禁用的（对于启用了 dnf 的系统，请在命令中用**dnf**替换**yum**）：

    ```sql
    $> yum repolist all | grep mysql
    ```

    要安装最新发布的最新 GA 系列，无需进行任何配置。要安装特定系列的最新发布而不是最新的 GA 系列，需要在运行安装命令之前禁用最新 GA 系列的子存储库并启用特定系列的子存储库。如果您的平台支持**yum-config-manager**，您可以通过以下命令执行此操作，这些命令禁用了 5.7 系列的子存储库并启用了 8.0 系列的子存储库：

    ```sql
    $> sudo yum-config-manager --disable mysql57-community
    $> sudo yum-config-manager --enable mysql80-community
    ```

    对于启用了 dnf 的平台：

    ```sql
    $> sudo dnf config-manager --disable mysql57-community
    $> sudo dnf config-manager --enable mysql80-community
    ```

    除了使用**yum-config-manager**或**dnf config-manager**命令外，您还可以通过手动编辑`/etc/yum.repos.d/mysql-community.repo`文件来选择一个发布系列。这是文件中发布系列子存储库的典型条目：

    ```sql
    [mysql57-community]
    name=MySQL 5.7 Community Server
    baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
           file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
    ```

    找到要配置的子存储库条目，并编辑`enabled`选项。指定`enabled=0`以禁用子存储库，或`enabled=1`以启用子存储库。例如，要安装 MySQL 8.0，请确保上述 MySQL 5.7 子存储库条目的`enabled=0`，并且对于 8.0 系列的条目，`enabled=1`：

    ```sql
    # Enable to use MySQL 8.0
    [mysql80-community]
    name=MySQL 8.0 Community Server
    baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2022
           file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
    ```

    您应该一次只启用一个发布系列的子存储库。当启用了多个发布系列的子存储库时，Yum 会使用最新的系列。

    通过运行以下命令并检查其输出来验证已启用和禁用正确的子存储库（对于启用 dnf 的系统，请在命令中用 **dnf** 替换 **yum**）：

    ```sql
    $> yum repolist enabled | grep mysql
    ```

1.  #### 禁用默认的 MySQL 模块

    （仅适用于 EL8 系统）基于 EL8 的系统（如 RHEL8 和 Oracle Linux 8）包含一个默认启用的 MySQL 模块。除非禁用此模块，否则会掩盖 MySQL 存储库提供的软件包。要禁用包含的模块并使 MySQL 存储库软件包可见，请使用以下命令（对于启用 dnf 的系统，请在命令中用 **dnf** 替换 **yum**）：

    ```sql
    $> sudo yum module disable mysql
    ```

1.  #### 安装 MySQL

    通过以下命令安装 MySQL（对于启用 dnf 的系统，请在命令中用 **dnf** 替换 **yum**）：

    ```sql
    $> sudo yum install mysql-community-server
    ```

    这将安装 MySQL 服务器的软件包（`mysql-community-server`）以及运行服务器所需组件的软件包，包括客户端的软件包（`mysql-community-client`）、客户端和服务器的常见错误消息和字符集的软件包（`mysql-community-common`）以及共享客户端库（`mysql-community-libs`）。

1.  #### 启动 MySQL 服务器

    使用��下命令启动 MySQL 服务器：

    ```sql
    $> systemctl start mysqld
    ```

    您可以使用以下命令检查 MySQL 服务器的状态：

    ```sql
    $> systemctl status mysqld
    ```

如果操作系统启用了 systemd，则应使用标准的 **systemctl** 命令（或者使用参数颠倒的 **service**）来管理 MySQL 服务器服务，如 **stop**、**start**、**status** 和 **restart**。`mysqld` 服务默认启用，并在系统重新启动时启动。有关更多信息，请参见 Section 2.5.9, “使用 systemd 管理 MySQL 服务器”。

在服务器初始启动时，假设服务器的数据目录为空，则会发生以下情况：

+   服务器已初始化。

+   SSL 证书和密钥文件在数据目录中生成。

+   `validate_password` 已安装并启用。

+   创建一个超级用户帐户 `'root'@'localhost`。设置并存储超级用户的密码在错误日志文件中。要显示它，请使用以下命令：

    ```sql
    $> sudo grep 'temporary password' /var/log/mysqld.log
    ```

    尽快使用生成的临时密码登录并为超级用户帐户设置自定义密码来更改根密码：

    ```sql
    $> mysql -uroot -p
    ```

    ```sql
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
    ```

    注意

    `validate_password` 默认安装。`validate_password` 实施的默认密码策略要求密码至少包含一个大写字母、一个小写字母、一个数字和一个特殊字符，并且密码总长度至少为 8 个字符。

有关后安装程序的更多信息，请参见 Section 2.9, “后安装设置和测试”。

注意

*基于 EL7 平台的兼容性信息：* 平台的本机软件存储库中的以下 RPM 软件包与从 MySQL Yum 存储库安装 MySQL 服务器的软件包不兼容。一旦您使用 MySQL Yum 存储库安装了 MySQL，您就无法安装这些软件包（反之亦然）。

+   akonadi-mysql

#### 使用 Yum 安装其他 MySQL 产品和组件

您可以使用 Yum 安装和管理 MySQL 的各个组件。其中一些组件托管在 MySQL Yum 存储库的子存储库中：例如，MySQL 连接器位于 MySQL 连接器社区子存储库中，而 MySQL Workbench 位于 MySQL 工具社区中。您可以使用以下命令列出来自 MySQL Yum 存储库的所有 MySQL 组件的软件包，适用于您平台（对于启用 dnf 的系统，请将命令中的 **yum** 替换为 **dnf**）：

```sql
$> sudo yum --disablerepo=\* --enablerepo='mysql*-community*' list available
```

使用以下命令安装您选择的任何软件包，将 *`package-name`* 替换为软件包的名称（对于启用 dnf 的系统，请将命令中的 **yum** 替换为 **dnf**）：

```sql
$> sudo yum install *package-name*
```

例如，在 Fedora 上安装 MySQL Workbench：

```sql
$> sudo dnf install mysql-workbench-community
```

要安装共享客户端库（对于启用 dnf 的系统，请将命令中的 **yum** 替换为 **dnf**）：

```sql
$> sudo yum install mysql-community-libs
```

#### 特定平台说明

ARM 支持

ARM 64 位（aarch64）在 Oracle Linux 7 上受支持，并需要 Oracle Linux 7 软件集合存储库（ol7_software_collections）。例如，要安装服务器：

```sql
$> yum-config-manager --enable ol7_software_collections
$> yum install mysql-community-server
```

注意

ARM 64 位（aarch64）在 MySQL 8.0.12 上受 Oracle Linux 7 支持。

已知限制

8.0.12 版本要求您在执行 `yum install` 步骤后通过执行 `ln -s /opt/oracle/oracle-armtoolset-1/root/usr/lib64 /usr/lib64/gcc7` 调整 *libstdc++7* 路径。

#### 使用 Yum 更新 MySQL

除了安装外，您还可以使用 MySQL Yum 存储库执行 MySQL 产品和组件的更新。有关详细信息，请参阅 第 3.8 节，“使用 MySQL Yum 存储库升级 MySQL”。
