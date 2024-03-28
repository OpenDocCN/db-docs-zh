# 2.7 在 Solaris 上安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/solaris-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/solaris-installation.html)

2.7.1 在 Solaris 上使用 Solaris PKG 安装 MySQL

注意

MySQL 8.0 支持 Solaris 11.4 及更高版本

Solaris 上的 MySQL 有多种不同的格式可用。

+   有关使用本机 Solaris PKG 格式安装的信息，请参阅第 2.7.1 节，“在 Solaris 上使用 Solaris PKG 安装 MySQL”。

+   要使用标准的 `tar` 二进制安装，请使用第 2.2 节，“在 Unix/Linux 上使用通用二进制文件安装 MySQL”中提供的注意事项。在安装之前或之后，查看本节末尾的 Solaris 特定注意事项。

注意

MySQL 5.7 依赖于 Oracle Developer Studio Runtime Libraries；但这不适用于 MySQL 8.0。

要获取 Solaris 的二进制 MySQL 分发包（tarball 或 PKG 格式），请访问[`dev.mysql.com/downloads/mysql/8.0.html`](https://dev.mysql.com/downloads/mysql/8.0.html)。

在 Solaris 上安装和使用 MySQL 时需要注意的其他事项：

+   如果想要使用 `mysql` 用户和组来运行 MySQL，请使用 **groupadd** 和 **useradd** 命令：

    ```sql
    groupadd mysql
    useradd -g mysql -s /bin/false mysql
    ```

+   如果在 Solaris 上使用二进制 tarball 分发安装 MySQL，因为 Solaris 的 **tar** 无法处理长文件名，请使用 GNU **tar** (**gtar**) 解压分发包。如果你的系统上没有 GNU **tar**，请使用以下命令安装：

    ```sql
    pkg install archiver/gnu-tar
    ```

+   你应该使用`forcedirectio`选项挂载任何你打算存储 `InnoDB` 文件的文件系统。（默认情况下，挂载是不带此选项的。）如果不这样做，在这个平台上使用 `InnoDB` 存储引擎时会导致性能显著下降。

+   如果希望 MySQL 自动启动，可以将 `support-files/mysql.server` 复制到 `/etc/init.d` 并创建一个名为 `/etc/rc3.d/S99mysql.server` 的符号链接。

+   如果有太多进程试图非常快速地连接到**mysqld**，你应该在 MySQL 日志中看到这个错误：

    ```sql
    Error in accept: Protocol error
    ```

    你可以尝试使用`--back_log=50`选项作为解决此问题的临时方法。

+   要在 Solaris 上配置核心文件的生成，应该使用 **coreadm** 命令。由于在 `setuid()` 应用程序上生成核心文件的安全性影响，默认情况下，Solaris 不支持 `setuid()` 程序的核心文件。但是，你可��使用 **coreadm** 修改这种行为。如果为当前用户启用 `setuid()` 核心文件，它们将以 600 模式生成，并由超级用户拥有。
