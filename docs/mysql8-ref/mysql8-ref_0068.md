# 2.5.4 使用 Oracle 提供的 RPM 软件包在 Linux 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/linux-installation-rpm.html`](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-rpm.html)

推荐在基于 RPM 的 Linux 发行版上安装 MySQL 的方法是使用 Oracle 提供的 RPM 软件包。有两个来源可以获取它们，适用于 MySQL 的社区版：

+   从 MySQL 软件仓库获取：

    +   MySQL Yum 仓库（详见 Section 2.5.1, “使用 MySQL Yum 仓库在 Linux 上安装 MySQL”）。

    +   MySQL SLES 仓库（详见 Section 2.5.3, “使用 MySQL SLES 仓库在 Linux 上安装 MySQL”）。

+   从[下载 MySQL 社区服务器](https://dev.mysql.com/downloads/mysql/)页面中获取，位于[MySQL 开发者区](https://dev.mysql.com/)。

注意

MySQL 的 RPM 发行版也由其他供应商提供。请注意，它们可能在功能、能力和约定（包括通信设置）方面与 Oracle 构建的不同，并且本手册中的安装说明不一定适用于它们。应该参考供应商的说明。

#### MySQL RPM 软件包

**表 2.9 MySQL 社区版的 RPM 软件包**

| 包名称 | 摘要 |
| --- | --- |
| `mysql-community-client` | MySQL 客户端应用程序和工具 |
| `mysql-community-client-plugins` | MySQL 客户端应用程序的共享插件 |
| `mysql-community-common` | 服务器和客户端库的通用文件 |
| `mysql-community-devel` | MySQL 数据库客户端应用程序的开发头文件和库 |
| `mysql-community-embedded-compat` | MySQL 服务器作为一个嵌入式库，兼容使用库版本 18 的应用程序 |
| `mysql-community-icu-data-files` | MySQL 打包的 ICU 数据文件，MySQL 正则表达式需要 |
| `mysql-community-libs` | MySQL 数据库客户端应用程序的共享库 |
| `mysql-community-libs-compat` | 用于先前 MySQL 安装的共享兼容性库；仅在平台支持先前的 MySQL 版本时存在 |
| `mysql-community-server` | 数据库服务器和相关工具 |
| `mysql-community-server-debug` | 调试服务器和插件二进制文件 |
| `mysql-community-test` | MySQL 服务器的测试套件 |
| `mysql-community` | 源代码 RPM 看起来类似于 mysql-community-8.0.36-1.el7.src.rpm，具体取决于所选的操作系统 |
| 其他*debuginfo* RPM | 有几个`debuginfo`软件包：mysql-community-client-debuginfo、mysql-community-libs-debuginfo、mysql-community-server-debug-debuginfo、mysql-community-server-debuginfo 和 mysql-community-test-debuginfo。 |
| 包名称 | 摘要 |

**表 2.10 MySQL 企业版的 RPM 软件包**

| 包名称 | 摘要 |
| --- | --- |
| `mysql-commercial-backup` | MySQL 企业备份（在 8.0.11 中添加） |
| `mysql-commercial-client` | MySQL 客户端应用程序和工具 |
| `mysql-commercial-client-plugins` | MySQL 客户端应用程序的共享插件 |
| `mysql-commercial-common` | 服务器和客户端库的通用文件 |
| `mysql-commercial-devel` | MySQL 数据库客户端应用程序的开发头文件和库 |
| `mysql-commercial-embedded-compat` | MySQL 服务器作为兼容性库嵌入式库，适用于使用库版本 18 的应用程序 |
| `mysql-commercial-icu-data-files` | MySQL 打包的 MySQL 正则表达式所需的 ICU 数据文件 |
| `mysql-commercial-libs` | MySQL 数据库客户端应用程序的共享库 |
| `mysql-commercial-libs-compat` | 用于先前 MySQL 安装的共享兼容性库；仅在平台支持先前的 MySQL 版本时存在。库的版本与您正在使用的发行版默认安装的库的版本相匹配。 |
| `mysql-commercial-server` | 数据库服务器和相关工具 |
| `mysql-commercial-test` | MySQL 服务器的测试套件 |
| 附加*debuginfo* RPMs | 有几个`debuginfo`包：mysql-commercial-client-debuginfo、mysql-commercial-libs-debuginfo、mysql-commercial-server-debug-debuginfo、mysql-commercial-server-debuginfo 和 mysql-commercial-test-debuginfo。 |
| 软件包名称 | 摘要 |

RPM 的完整名称具有以下语法：

```sql
*packagename*-*version*-*distribution*-*arch*.rpm
```

*`distribution`*和*`arch`*值表示构建软件包的 Linux 发行版和处理器类型。请参阅下表以获取发行标识符的列表：

**表 2.11 MySQL Linux RPM 软件包分发标识符**

| 发行值 | 预期用途 |
| --- | --- |
| el*`{version}`*，其中*`{version}`*是主要的 Enterprise Linux 版本，例如`el8` | 基于 EL6（8.0）、EL7、EL8 和 EL9 的平台（例如，Oracle Linux、Red Hat Enterprise Linux 和 CentOS 的相应版本） |
| fc*`{version}`*，其中*`{version}`*是主要的 Fedora 版本，例如`fc37` | Fedora 38 和 39 |
| `sles12` | SUSE Linux 企业服务器 12 |

要查看 RPM 软件包中的所有文件（例如，`mysql-community-server`），请使用以下命令：

```sql
$> rpm -qpl mysql-community-server-*version*-*distribution*-*arch*.rpm
```

*本节其余部分的讨论仅适用于直接从 Oracle 下载的 RPM 软件包进行安装过程，而不是通过 MySQL 存储库。*

一些软件包之间存在依赖关系。如果您计划安装许多软件包，您可能希望下载 RPM 捆绑包**tar**文件，其中包含上述所有 RPM 软件包，这样您就不需要单独下载它们。

在大多数情况下，您需要安装 `mysql-community-server`、`mysql-community-client`、`mysql-community-client-plugins`、`mysql-community-libs`、`mysql-community-icu-data-files`、`mysql-community-common` 和 `mysql-community-libs-compat` 软件包才能获得一个功能齐全的标准 MySQL 安装。要执行这样的标准、基本安装，请转到包含所有这些软件包的文件夹（最好不要包含其他名称类似的 RPM 软件包），然后执行以下命令：

```sql
$> sudo yum install mysql-community-{server,client,client-plugins,icu-data-files,common,libs}-*
```

在 SLES 中用 **zypper** 替换 **yum**，在 Fedora 中用 **dnf** 替换。

尽管最好使用像 **yum** 这样的高级包管理工具来安装软件包，但喜欢直接使用 **rpm** 命令的用户可以将 **yum install** 命令替换为 **rpm -Uvh** 命令；然而，使用 **rpm -Uvh** 会使安装过程更容易失败，因为安装过程可能会遇到潜在的依赖性问题。

要仅安装客户端程序，您可以在要安装的软件包列表中跳过 `mysql-community-server`；执行以下命令：

```sql
$> sudo yum install mysql-community-{client,client-plugins,common,libs}-*
```

在 SLES 中用 **zypper** 替换 **yum**，在 Fedora 中用 **dnf** 替换。

使用 RPM 软件包进行标准的 MySQL 安装会在系统目录下创建文件和资源，如下表所示。

**表 2.12 来自 MySQL Developer Zone 的 Linux RPM 软件包的 MySQL 安装布局**

| 文件或资源 | 位置 |
| --- | --- |
| 客户端程序和脚本 | `/usr/bin` |
| **mysqld** 服务器 | `/usr/sbin` |
| 配置文件 | `/etc/my.cnf` |
| 数据目录 | `/var/lib/mysql` |
| 错误日志文件 | 对于 RHEL、Oracle Linux、CentOS 或 Fedora 平台：`/var/log/mysqld.log`对于 SLES：`/var/log/mysql/mysqld.log` |
| `secure_file_priv` 的值 | `/var/lib/mysql-files` |
| System V init script | 对于 RHEL、Oracle Linux、CentOS 或 Fedora 平台：`/etc/init.d/mysqld`对于 SLES：`/etc/init.d/mysql` |
| Systemd 服务 | 对于 RHEL、Oracle Linux、CentOS 或 Fedora 平台：`mysqld`对于 SLES：`mysql` |
| Pid 文件 | `/var/run/mysql/mysqld.pid` |
| Socket | `/var/lib/mysql/mysql.sock` |
| 密钥环目录 | `/var/lib/mysql-keyring` |
| Unix 手册页 | `/usr/share/man` |
| 包含（头）文件 | `/usr/include/mysql` |
| 库文件 | `/usr/lib/mysql` |
| 其他支持文件（例如，错误消息和字符集文件） | `/usr/share/mysql` |
| 文件或资源 | 位置 |

安装还会在系统上创建一个名为 `mysql` 的用户和一个名为 `mysql` 的组。

注意

+   使用`useradd`命令的`-r`和`-s /bin/false`选项创建`mysql`用户，以便它没有登录服务器主机的权限（有关详细信息，请参见创建 mysql 用户和组）。要在您的操作系统上切换到`mysql`用户，请使用`su`命令的`--shell=/bin/bash`选项：

    ```sql
    su - mysql --shell=/bin/bash
    ```

+   使用旧软件包安装之前版本的 MySQL 可能已经创建了名为`/usr/my.cnf`的配置文件。强烈建议您检查文件的内容，并将所需的设置迁移到文件`/etc/my.cnf`中，然后删除`/usr/my.cnf`。

MySQL 在安装过程结束时不会自动启动。对于 Red Hat Enterprise Linux、Oracle Linux、CentOS 和 Fedora 系统，请使用以下命令启动 MySQL：

```sql
$> systemctl start mysqld
```

对于 SLES 系统，命令是相同的，但服务名称不同：

```sql
$> systemctl start mysql
```

如果操作系统启用了 systemd，则应使用标准的**systemctl**（或者使用参数颠倒的**service**）命令来管理 MySQL 服务器服务，例如**stop**、**start**、**status**和**restart**。`mysqld`服务默认启用，并在系统重新启动时启动。请注意，在 systemd 平台上可能会有一些不同的工作方式：例如，更改数据目录的位置可能会导致问题。有关更多信息，请参见 Section 2.5.9，“使用 systemd 管理 MySQL 服务器”。

在使用 RPM 和 DEB 软件包进行升级安装时，如果 MySQL 服务器在升级发生时正在运行，则 MySQL 服务器将被停止，升级将进行，然后 MySQL 服务器将被重新启动。一个例外：如果在升级过程中还更改了版本（例如从社区版到商业版，反之亦然），则 MySQL 服务器不会重新启动。

在服务器初始启动时，假设服务器的数据目录为空时，会发生以下情况：

+   服务器已初始化。

+   在数据目录中生成 SSL 证书和密钥文件。

+   `validate_password`已安装并启用。

+   创建超级用户帐户`'root'@'localhost'`。设置超级用户的密码并将其存储在错误日志文件中。要显示密码，请对于 RHEL、Oracle Linux、CentOS 和 Fedora 系统使用以下命令：

    ```sql
    $> sudo grep 'temporary password' /var/log/mysqld.log
    ```

    对于 SLES 系统，请使用以下命令：

    ```sql
    $> sudo grep 'temporary password' /var/log/mysql/mysqld.log
    ```

    下一步是使用生成的临时密码登录并为超级用户帐户设置自定义密码：

```sql
$> mysql -uroot -p
```

```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```

注意

默认情况下安装了`validate_password`。由`validate_password`实施的默认密码策略要求密码至少包含一个大写字母，一个小写字母，一个数字和一个特殊字符，并且密码总长度至少为 8 个字符。

如果在安装过程中出现问题，您可能会在错误日志文件`/var/log/mysqld.log`中找到调试信息。

对于一些 Linux 发行版，可能需要增加可用文件描述符数量的限制以供**mysqld**使用。请参阅 Section B.3.2.16, “File Not Found and Similar Errors”。

**从多个 MySQL 版本安装客户端库。** 可以安装多个客户端库版本，例如，如果您希望与链接到先前库的旧应用程序保持兼容性。要安装旧的客户端库，请使用**rpm**的`--oldpackage`选项。例如，要在具有来自 MySQL 8.0 的`libmysqlclient.21`的 EL6 系统上安装`mysql-community-libs-5.5`，请使用以下命令：

```sql
$> rpm --oldpackage -ivh mysql-community-libs-5.5.50-2.el6.x86_64.rpm
```

**调试包。** MySQL Server 的一个特殊变体编译了调试包并包含在服务器 RPM 包中。它执行调试和内存分配检查，并在服务器运行时生成一个跟踪文件。要使用该调试版本，请使用`/usr/sbin/mysqld-debug`启动 MySQL，而不是像服务或使用`/usr/sbin/mysqld`启动。请参阅 Section 7.9.4, “The DBUG Package”以了解您可以使用的调试选项。

注意

调试版本的默认插件目录从 MySQL 8.0.4 中的`/usr/lib64/mysql/plugin`更改为`/usr/lib64/mysql/plugin/debug`。以前，必须将`plugin_dir`更改为`/usr/lib64/mysql/plugin/debug`以供调试版本使用。

**从源 SRPM 重新构建 RPM。** MySQL 的源代码 SRPM 包可以下载。它们可以原样使用，使用标准的**rpmbuild**工具链重新构建 MySQL 的 RPM 包。
