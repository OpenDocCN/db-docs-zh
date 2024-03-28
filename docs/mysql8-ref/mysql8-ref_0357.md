# 8.1.5 如何以普通用户身份运行 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/changing-mysql-user.html`](https://dev.mysql.com/doc/refman/8.0/en/changing-mysql-user.html)

在 Windows 上，您可以使用普通用户帐户将服务器作为 Windows 服务运行。

在 Linux 上，如果使用 MySQL 存储库或 RPM 软件包进行安装，则应该由本地`mysql`操作系统用户启动 MySQL 服务器**mysqld**。使用其他操作系统用户启动不受 MySQL 存储库包含的 init 脚本支持。

在 Unix（或 Linux 使用`tar.gz`软件包进行安装时），MySQL 服务器**mysqld**可以由任何用户启动和运行。但是，出于安全原因，您应避免以 Unix `root`用户身份运行服务器。要将**mysqld**更改为以普通非特权 Unix 用户*`user_name`*运行，您必须执行以下操作：

1.  如果服务器正在运行，请停止它（使用**mysqladmin shutdown**）。

1.  更改数据库目录和文件，以便*`user_name`*有权限在其中读取和写入文件（您可能需要以 Unix `root`用户的身份执行此操作）：

    ```sql
    $> chown -R *user_name* */path/to/mysql/datadir*
    ```

    如果不这样做，服务器在以*`user_name`*运行时无法访问数据库或表。

    如果 MySQL 数据目录中的目录或文件是符号链接，则`chown -R`可能不会为您跟随符号链接。如果没有，请您也必须跟随这些链接并更改它们指向的目录和文件。

1.  以用户*`user_name`*启动服务器。另一种选择是以 Unix `root`用户启动**mysqld**，并使用`--user=*`user_name`*`选项。**mysqld**启动后，然后在接受任何连接之前切换为以 Unix 用户*`user_name`*运行。

1.  要在系统启动时自动以给定用户启动服务器，请通过在`/etc/my.cnf`选项文件的`[mysqld]`组或服务器数据目录中的`my.cnf`选项文件中添加`user`选项来指定用户名。例如：

    ```sql
    [mysqld]
    user=*user_name*
    ```

如果您的 Unix 机器本身没有受到保护，您应该在授权表中为 MySQL `root`帐户分配密码。否则，任何在该机器上具有登录帐户的用户都可以使用`--user=root`选项运行**mysql**客户端并执行任何操作。（在任何情况下为 MySQL 帐户分配密码都是个好主意，但特别是在服务器主机上存在其他登录帐户时更是如此。）请参见 Section 2.9.4, “Securing the Initial MySQL Account”。
