# 6.3.3 mysql.server — MySQL Server Startup Script

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-server.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-server.html)

Unix 和类 Unix 系统上的 MySQL 发行版包含一个名为**mysql.server**的脚本，它使用**mysqld_safe**启动 MySQL 服务器。它可以在使用 System V 风格运行目录启动和停止系统服务的系统上使用，例如 Linux 和 Solaris。它还被 macOS 的 MySQL 启动项使用。

**mysql.server**是 MySQL 源代码树中使用的脚本名称。安装后的名称可能不同（例如，**mysqld**或**mysql**）。在下面的讨论中，根据您的系统调整名称**mysql.server**。

注意

对于某些 Linux 平台，从 RPM 或 Debian 软件包安装 MySQL 包括 systemd 支持以管理 MySQL 服务器的启动和关闭。在这些平台上，不安装**mysql.server**和**mysqld_safe**，因为它们是不必要的。有关更多信息，请参见 Section 2.5.9, “使用 systemd 管理 MySQL 服务器”。

要手动启动或停止服务器，可以使用**mysql.server**脚本，在命令行中使用`start`或`stop`参数调用它：

```sql
mysql.server start
mysql.server stop
```

**mysql.server**将位置更改为 MySQL 安装目录，然后调用**mysqld_safe**。要以某个特定用户身份运行服务器，请在全局`/etc/my.cnf`选项文件的`[mysqld]`组中添加适当的`user`选项，如本节后面所示。（如果您在非标准位置安装了 MySQL 的二进制发行版，则可能必须编辑**mysql.server**。修改它以在运行**mysqld_safe**之前将位置更改为正确的目录。如果这样做，您修改过的**mysql.server**版本可能会在将来升级 MySQL 时被覆盖；请制作一个您可以重新安装的编辑过的版本的副本。）

**mysql.server stop** 通过向服务器发送信号来停止服务器。你也可以通过执行**mysqladmin shutdown**来手动停止服务器。

要在服务器上自动启动和停止 MySQL，你必须在`/etc/rc*`文件的适当位置添加启动和停止命令：

+   如果你使用 Linux 服务器 RPM 包（`MySQL-server-*`VERSION`*.rpm`），或者本机 Linux 包安装，**mysql.server** 脚本可能会安装在`/etc/init.d`目录中，名称为`mysqld`或`mysql`。有关 Linux RPM 包的更多信息，请参阅第 2.5.4 节，“使用 Oracle 的 RPM 包在 Linux 上安装 MySQL”。

+   如果你从源分发或使用不会自动安装**mysql.server**的二进制分发格式安装 MySQL，你可以手动安装该脚本。它可以在 MySQL 安装目录的`support-files`目录下或 MySQL 源代码树中找到。将脚本复制到`/etc/init.d`目录中，并命名为**mysql**，然后使其可执行：

    ```sql
    cp mysql.server /etc/init.d/mysql
    chmod +x /etc/init.d/mysql
    ```

    安装脚本后，激活它以在系统启动时运行所需的命令取决于你的操作系统。在 Linux 上，你可以使用 **chkconfig**：

    ```sql
    chkconfig --add mysql
    ```

    在一些 Linux 系统上，似乎还需要以下命令才能完全启用**mysql** 脚本：

    ```sql
    chkconfig --level 345 mysql on
    ```

+   在 FreeBSD 上，启动脚本通常应放在`/usr/local/etc/rc.d/`中。将`mysql.server`脚本安装为`/usr/local/etc/rc.d/mysql.server.sh`以启用自动启动。`rc(8)`手册页指出，此目录中的脚本仅在其基本名称与`*.sh` shell 文件名模式匹配时才会执行。目录中存在的任何其他文件或目录都会被静默忽略。

+   作为前述设置的替代方案，一些操作系统还使用`/etc/rc.local`或`/etc/init.d/boot.local`在启动时启动其他服务。要使用这种方法启动 MySQL，将类似以下命令附加到适当的启动文件中：

    ```sql
    /bin/sh -c 'cd /usr/local/mysql; ./bin/mysqld_safe --user=mysql &'
    ```

+   对于其他系统，请查阅操作系统文档以了解如何安装启动脚本。

**mysql.server** 从选项文件的`[mysql.server]`和`[mysqld]`部分读取选项。为了向后兼容，它还会读取`[mysql_server]`部分，但为了保持最新，你应该将这些部分重命名为`[mysql.server]`。

您可以在全局 `/etc/my.cnf` 文件中为 **mysql.server** 添加选项。典型的 `my.cnf` 文件可能如下所示：

```sql
[mysqld]
datadir=/usr/local/mysql/var
socket=/var/tmp/mysql.sock
port=3306
user=mysql

[mysql.server]
basedir=/usr/local/mysql
```

**mysql.server** 脚本支持以下表中显示的选项。如果指定，它们*必须*放在选项文件中，而不是在命令行上。**mysql.server** 仅支持 `start` 和 `stop` 作为命令行参数。

**表 6.8 mysql.server 选项文件选项**

| 选项名称 | 描述 | 类型 |
| --- | --- | --- |
| `basedir` | MySQL 安装目录的路径 | 目录名 |
| `datadir` | MySQL 数据目录的路径 | 目录名 |
| `pid-file` | 服务器应将其进程 ID 写入的文件 | 文件名 |
| `service-startup-timeout` | 等待服务器启动的时间 | 整数 |

+   `basedir=*`dir_name`*`

    MySQL 安装目录的路径。

+   `datadir=*`dir_name`*`

    MySQL 数据目录的路径。

+   `pid-file=*`file_name`*`

    服务器应将其进程 ID 写入的文件的路径名。除非给定绝对路径名以指定不同目录，否则服务器将在数据目录中创建该文件。

    如果未提供此选项，**mysql.server** 使用默认值 `*`host_name`*.pid`。传递给 **mysqld_safe** 的 PID 文件值会覆盖在 `[mysqld_safe]` 选项文件组中指定的任何值。因为 **mysql.server** 读取 `[mysqld]` 选项文件组而不是 `[mysqld_safe]` 组，您可以确保在从 **mysql.server** 调用时与手动调用时放置相同的 `pid-file` 设置在 `[mysqld_safe]` 和 `[mysqld]` 组中，以确保 **mysqld_safe** 获得相同的值。

+   `service-startup-timeout=*`seconds`*`

    等待服务器启动确认的秒数。如果服务器在此时间内未启动，**mysql.server** 将以错误退出。默认值为 900。值为 0 表示根本不等待启动。负值表示永远等待（无超时）。
