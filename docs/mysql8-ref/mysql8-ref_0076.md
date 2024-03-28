# 2.5.9 使用 systemd 管理 MySQL 服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-systemd.html`](https://dev.mysql.com/doc/refman/8.0/en/using-systemd.html)

如果您在以下 Linux 平台上使用 RPM 或 Debian 软件包安装 MySQL，则服务器的启动和关闭由 systemd 管理：

+   RPM 软件包平台：

    +   企业 Linux 变体版本 7 及更高版本

    +   SUSE Linux Enterprise Server 12 及更高版本

    +   Fedora 29 及更高版本

+   Debian 家族平台：

    +   Debian 平台

    +   Ubuntu 平台

如果您在使用 systemd 的平台上从通用二进制发行版安装 MySQL，则可以按照 MySQL 8.0 安全部署指南中提供的后安装设置部分的说明手动配置 MySQL 的 systemd 支持。

如果您在使用 systemd 的平台上从源代码发行版安装 MySQL，则可以通过使用 `-DWITH_SYSTEMD=1` **CMake** 选项配置发行版以获得 MySQL 的 systemd 支持。请参阅第 2.8.7 节，“MySQL 源代码配置选项”。

以下讨论涵盖了这些主题：

+   systemd 概述

+   为 MySQL 配置 systemd

+   使用 systemd 配置多个 MySQL 实例

+   从 mysqld_safe 迁移到 systemd

注意

在为 MySQL 安装了 systemd 支持的平台上，诸如**mysqld_safe**和 System V 初始化脚本等脚本是不必要的，也不会被安装。例如，**mysqld_safe**可以处理服务器重启，但 systemd 提供了相同的功能，并且以与其他服务管理一致的方式进行，而不是使用特定于应用程序的程序。

在使用 systemd 进行服务器管理的平台上不使用**mysqld_safe**的一个影响是，不支持在选项文件中使用 `[mysqld_safe]` 或 `[safe_mysqld]` 部分，可能会导致意外行为。

因为 systemd 在安装了 MySQL systemd 支持的平台上具有管理多个 MySQL 实例的能力，所以**mysqld_multi**和 **mysqld_multi.server** 是不必要的，也不会被安装。

#### systemd 概述

systemd 提供自动 MySQL 服务器启动和关闭。它还通过**systemctl**命令启用手动服务器管理。例如：

```sql
$> systemctl *{start|stop|restart|status}* mysqld
```

或者，使用**service**命令（参数颠倒），该命令与 System V 系统兼容：

```sql
$> service mysqld *{start|stop|restart|status}*
```

注意

对于**systemctl**命令（以及替代的**service**命令），如果 MySQL 服务名称不是`mysqld`，则使用适当的名称。例如，在基于 Debian 和 SLES 系统上使用`mysql`而不是`mysqld`。

systemd 的支持包括这些文件：

+   `mysqld.service`（RPM 平台），`mysql.service`（Debian 平台）：systemd 服务单元配置文件，包含有关 MySQL 服务的详细信息。

+   `mysqld@.service`（RPM 平台），`mysql@.service`（Debian 平台）：类似于`mysqld.service`或`mysql.service`，但用于管理多个 MySQL 实例。

+   `mysqld.tmpfiles.d`：包含支持`tmpfiles`功能信息的文件。此文件安装为`mysql.conf`。

+   `mysqld_pre_systemd`（RPM 平台），`mysql-system-start`（Debian 平台）：单元文件的支持脚本。此脚本仅在错误日志位置匹配模式（对于 RPM 平台为`/var/log/mysql*.log`，对于 Debian 平台为`/var/log/mysql/*.log`）时协助创建错误日志文件。在其他情况下，错误日志目录必须可写，或者错误日志必须存在并对运行**mysqld**进程的用户可写。

#### 配置 MySQL 的 systemd

要为 MySQL 添加或更改 systemd 选项，可以使用以下方法：

+   使用本地化的 systemd 配置文件。

+   安排 systemd 为 MySQL 服务器进程设置环境变量。

+   设置`MYSQLD_OPTS` systemd 变量。

要使用本地化的 systemd 配置文件，请创建`/etc/systemd/system/mysqld.service.d`目录（如果不存在）。在该目录中，创建一个包含列出所需设置的`[Service]`部分的文件。例如：

```sql
[Service]
LimitNOFILE=*max_open_files*
Nice=*nice_level*
LimitCore=*core_file_limit*
Environment="LD_PRELOAD=*/path/to/malloc/library*"
Environment="TZ=*time_zone_setting*"
```

此处讨论使用`override.conf`作为此文件的名称。较新版本的 systemd 支持以下命令，该命令打开编辑器并允许您编辑文件：

```sql
systemctl edit mysqld  # RPM platforms
systemctl edit mysql   # Debian platforms
```

每当创建或更改`override.conf`时，重新加载 systemd 配置，然后告诉 systemd 重新启动 MySQL 服务：

```sql
systemctl daemon-reload
systemctl restart mysqld  # RPM platforms
systemctl restart mysql   # Debian platforms
```

对于某些参数，必须使用`override.conf`配置方法，而不是在 MySQL 选项文件的`[mysqld]`、`[mysqld_safe]`或`[safe_mysqld]`组中的设置：

+   对于某些参数，必须使用`override.conf`，因为 systemd 本身必须知道它们的值，而不能读取 MySQL 选项文件来获取它们。

+   指定值的参数，否则只能使用已知于**mysqld_safe**的选项设置，必须使用 systemd 指定，因为没有对应的**mysqld**参数。

有关使用 systemd 而不是**mysqld_safe**的更多信息，请参阅从 mysqld_safe 迁移到 systemd。

您可以在`override.conf`中设置以下参数：

+   要设置 MySQL 服务器可用的文件描述符数量，请在`override.conf`中使用`LimitNOFILE`，而不是**mysqld**的`open_files_limit`系统变量或**mysqld_safe**的`--open-files-limit`选项。

+   要设置最大核心文件大小，请在`override.conf`中使用`LimitCore`，而不是`--core-file-size`选项用于**mysqld_safe**。

+   要为 MySQL 服务器设置调度优先级，请在`override.conf`中使用`Nice`，而不是`--nice`选项用于**mysqld_safe**。

一些 MySQL 参数使用环境变量进行配置：

+   `LD_PRELOAD`: 如果 MySQL 服务器应该使用特定的内存分配库，请设置此变量。

+   `NOTIFY_SOCKET`: 此环境变量指定**mysqld**用于与 systemd 通信启动完成和服务状态更改通知的套接字。当启动**mysqld**服务时，systemd 会设置它。**mysqld**服务读取变量设置并写入定义的位置。

    在 MySQL 8.0 中，**mysqld**使用`Type=notify`进程启动类型。 （MySQL 5.7 中使用`Type=forking`。）使用`Type=notify`，systemd 会自动配置套接字文件并导出路径到`NOTIFY_SOCKET`环境变量。

+   `TZ`: 设置此变量以指定服务器的默认时区。

有多种方法可以指定由 systemd 管理的 MySQL 服务器进程使用的环境变量值：

+   在`override.conf`文件中使用`Environment`行。有关语法，请参见前面讨论中描述如何使用此文件的示例。

+   在`/etc/sysconfig/mysql`文件中指定值（如果不存在，请创建该文件）。使用以下语法分配值：

    ```sql
    LD_PRELOAD=*/path/to/malloc/library*
    TZ=*time_zone_setting*
    ```

    修改`/etc/sysconfig/mysql`后，重新启动服务器以使更改生效：

    ```sql
    systemctl restart mysqld  # RPM platforms
    systemctl restart mysql   # Debian platforms
    ```

要为**mysqld**指定选项而不直接修改 systemd 配置文件，请设置或取消设置`MYSQLD_OPTS` systemd 变量。例如：

```sql
systemctl set-environment MYSQLD_OPTS="--general_log=1"
systemctl unset-environment MYSQLD_OPTS
```

`MYSQLD_OPTS` 也可以在 `/etc/sysconfig/mysql` 文件中设置。

修改 systemd 环境后，重新启动服务器以使更改生效：

```sql
systemctl restart mysqld  # RPM platforms
systemctl restart mysql   # Debian platforms
```

对于使用 systemd 的平台，如果数据目录在服务器启动时为空，则会初始化数据目录。如果数据目录是一个暂时消失的远程挂载点，这可能会成为一个问题：挂载点看起来像一个空的数据目录，然后会被初始化为一个新的数据目录。要抑制此自动初始化行为，请在 `/etc/sysconfig/mysql` 文件中指定以下行（如果文件不存在，请创建文件）：

```sql
NO_INIT=true
```

#### 使用 systemd 配置多个 MySQL 实例

本节描述了如何为多个 MySQL 实例配置 systemd。

注意

因为 systemd 在安装了 systemd 支持的平台上具有管理多个 MySQL 实例的能力，所以 **mysqld_multi** 和 **mysqld_multi.server** 是不必要的，也不会安装。

要使用多实例功能，修改 `my.cnf` 选项文件以包含每个实例的关键选项配置。这些文件位置通常如下：

+   `/etc/my.cnf` 或 `/etc/mysql/my.cnf`（RPM 平台）

+   `/etc/mysql/mysql.conf.d/mysqld.cnf`（Debian 平台）

例如，要管理名为 `replica01` 和 `replica02` 的两个实例，请在选项文件中添加类似以下内容：

RPM 平台：

```sql
[mysqld@replica01]
datadir=/var/lib/mysql-replica01
socket=/var/lib/mysql-replica01/mysql.sock
port=3307
log-error=/var/log/mysqld-replica01.log

[mysqld@replica02]
datadir=/var/lib/mysql-replica02
socket=/var/lib/mysql-replica02/mysql.sock
port=3308
log-error=/var/log/mysqld-replica02.log
```

Debian 平台：

```sql
[mysqld@replica01]
datadir=/var/lib/mysql-replica01
socket=/var/lib/mysql-replica01/mysql.sock
port=3307
log-error=/var/log/mysql/replica01.log

[mysqld@replica02]
datadir=/var/lib/mysql-replica02
socket=/var/lib/mysql-replica02/mysql.sock
port=3308
log-error=/var/log/mysql/replica02.log
```

这里显示的副本名称使用 `@` 作为分隔符，因为这是 systemd 支持的唯一分隔符。

然后实例将由正常的 systemd 命令管理，例如：

```sql
systemctl start mysqld@replica01
systemctl start mysqld@replica02
```

要在启动时启用实例运行，请执行以下操作：

```sql
systemctl enable mysqld@replica01
systemctl enable mysqld@replica02
```

也支持使用通配符。例如，以下命令显示所有副本实例的状态：

```sql
systemctl status 'mysqld@replica*'
```

对于在同一台机器上管理多个 MySQL 实例，systemd 自动使用不同的单元文件：

+   `mysqld@.service` 而不是 `mysqld.service`（RPM 平台）

+   `mysql@.service` 而不是 `mysql.service`（Debian 平台）

在单元文件中，`%I` 和 `%i` 引用在 `@` 标记后传递的参数，并用于管理特定实例。对于这样的命令：

```sql
systemctl start mysqld@replica01
```

systemd 使用类似以下命令启动服务器：

```sql
mysqld --defaults-group-suffix=@%I ...
```

结果是 `[server]`、`[mysqld]` 和 `[mysqld@replica01]` 选项组将被读取并用于该服务的实例。

注意

在 Debian 平台上，AppArmor 阻止服务器读取或写入 `/var/lib/mysql-replica*`，或者除了默认位置之外的任何其他位置。要解决这个问题，您必须自定义或禁用 `/etc/apparmor.d/usr.sbin.mysqld` 中的配置文件。

注意

在 Debian 平台上，MySQL 卸载的打包脚本目前无法处理 `mysqld@` 实例。在删除或升级软件包之前，您必须首先手动停止任何额外的实例。

#### 从 mysqld_safe 迁移到 systemd

因为在使用 systemd 管理 MySQL 的平台上没有安装 **mysqld_safe**，先前为该程序指定的选项（例如，在 `[mysqld_safe]` 或 `[safe_mysqld]` 选项组中）必须以另一种方式指定：

+   一些 **mysqld_safe** 选项也被 **mysqld** 理解，并且可以从 `[mysqld_safe]` 或 `[safe_mysqld]` 选项组移动到 `[mysqld]` 组。这不包括 `--pid-file`，`--open-files-limit`，或 `--nice`。要指定这些选项，请使用之前描述的 `override.conf` systemd 文件。

    注意

    在 systemd 平台上，不支持使用 `[mysqld_safe]` 和 `[safe_mysqld]` 选项组，并且可能导致意外行为。

+   对于一些 **mysqld_safe** 选项，有替代的 **mysqld** 过程。例如，启用 `syslog` 记录的 **mysqld_safe** 选项是 `--syslog`，已被弃用。要将错误日志输出写入系统日志，请使用 Section 7.4.2.8, “Error Logging to the System Log” 中的说明。

+   **mysqld_safe** 不理解的选项可以在 `override.conf` 或环境变量中指定。例如，使用 **mysqld_safe**，如果服务器应该使用特定的内存分配库，则使用 `--malloc-lib` 选项指定。对于使用 systemd 管理服务器的安装，安排设置 `LD_PRELOAD` 环境变量，如之前描述的那样。
