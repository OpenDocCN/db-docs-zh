> 原文：[`dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html`](https://dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html)

#### B.3.3.2 如何重置根密码

如果您从未为 MySQL 分配过`root`密码，则服务器根本不需要密码以`root`身份连接。但是，这是不安全的。有关分配密码的说明，请参阅 Section 2.9.4，“Securing the Initial MySQL Account”。

如果您知道`root`密码并想要更改它，请参阅 Section 15.7.1.1，“ALTER USER Statement”和 Section 15.7.1.10，“SET PASSWORD Statement”。

如果您之前分配了`root`密码但忘记了，您可以分配一个新密码。以下部分提供了适用于 Windows 和 Unix 以及类 Unix 系统的说明，以及适用于任何系统的通用说明。

##### B.3.3.2.1 重置根密码：Windows 系统

在 Windows 上，使用以下过程重置 MySQL `'root'@'localhost'`帐户的密码。要更改具有不同主机名部分的`root`帐户的密码，请修改说明以使用该主机名。

1.  以管理员身份登录到系统。

1.  如果 MySQL 服务器正在运行，请停止它。对于作为 Windows 服务运行的服务器，请转到服务管理器：从开始菜单中，选择控制面板，然后选择管理工具，然后选择服务。在列表中找到 MySQL 服务并停止它。

    如果您的服务器未作为服务运行，您可能需要使用任务管理器强制停止它。

1.  创建一个包含密码分配语句的文本文件，将密码替换为您想要使用的密码。

    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
    ```

1.  保存文件。此示例假定您将文件命名为`C:\mysql-init.txt`。

1.  打开控制台窗口以进入命令提示符：从开始菜单中，选择运行，然后输入**cmd**作为要运行的命令。

1.  使用`init_file`系统变量设置启动时执行文件的名称启动 MySQL 服务器（请注意，选项值中的反斜杠是双倍的）：

    ```sql
    C:\> cd "C:\Program Files\MySQL\MySQL Server 8.0\bin"
    C:\> mysqld --init-file=C:\\mysql-init.txt
    ```

    如果您将 MySQL 安装到不同位置，请相应调整**cd**命令。

    服务器在启动时执行由`init_file`系统变量命名的文件的内容，更改`'root'@'localhost'`帐户密码。

    要使服务器输出显示在控制台窗口而不是日志文件中，请将`--console`选项添加到**mysqld**命令中。

    如果您使用 MySQL 安装向导安装 MySQL，���可能需要指定一个`--defaults-file`选项。例如：

    ```sql
    C:\> mysqld
             --defaults-file="C:\\ProgramData\\MySQL\\MySQL Server 8.0\\my.ini"
             --init-file=C:\\mysql-init.txt
    ```

    可以使用服务管理器找到适当的 `--defaults-file` 设置：从开始菜单中，选择控制面板，然后选择管理工具，然后选择服务。在列表中找到 MySQL 服务，右键单击它，然后选择 `属性` 选项。`可执行路径` 字段包含 `--defaults-file` 设置。

1.  服务器成功启动后，删除 `C:\mysql-init.txt`。

现在您应该能够使用新密码连接到 MySQL 服务器作为 `root`。停止 MySQL 服务器并正常重新启动。如果您将服务器作为服务运行，请从 Windows 服务窗口启动它。如果您手动启动服务器，请使用您通常使用的命令。

##### B.3.3.2.2 重置根密码：Unix 和类 Unix 系统

在 Unix 上，使用以下步骤重置 MySQL `'root'@'localhost'` 帐户的密码。要更改具有不同主机名部分的 `root` 帐户的密码，请修改指令以使用该主机名。

说明假定您从通常用于运行 MySQL 的 Unix 登录帐户启动 MySQL 服务器。例如，如果您使用 `mysql` 登录帐户运行服务器，则在使用说明之前应以 `mysql` 登录。或者，您可以以 `root` 身份登录，但在这种情况下，*必须* 使用 `--user=mysql` 选项启动 **mysqld**。如果以 `root` 身份启动服务器而不使用 `--user=mysql`，服务器可能会在数据目录中创建属于 `root` 的文件，例如日志文件，这可能会导致未来服务器启动时的权限相关问题。如果发生这种情况，您必须将文件的所有权更改为 `mysql` 或删除这些文件。

1.  以 MySQL 服务器运行的 Unix 用户身份登录到系统上（例如，`mysql`）。

1.  如果 MySQL 服务器正在运行，请停止它。找到包含服务器进程 ID 的 `.pid` 文件。此文件的确切位置和名称取决于您的发行版、主机名和配置。常见位置包括 `/var/lib/mysql/`、`/var/run/mysqld/` 和 `/usr/local/mysql/data/`。通常，文件名具有 `.pid` 扩展名，并以 `mysqld` 或您系统的主机名开头。

    通过向 **mysqld** 进程发送正常的 `kill` 命令（而不是 `kill -9`）来停止 MySQL 服务器。在以下命令中使用 `.pid` 文件的实际路径名：

    ```sql
    $> kill `cat /mysql-data-directory/host_name.pid`
    ```

    使用 `cat` 命令时请使用反引号（而不是前引号）。这会导致 `cat` 的输出被替换为 `kill` 命令。

1.  创建一个包含密码分配语句的文本文件，将密码替换为您想要使用的密码。

    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
    ```

1.  保存文件。此示例假定您将文件命名为`/home/me/mysql-init`。该文件包含密码，因此不要将其保存在其他用户可以读取的位置。如果您没有以`mysql`（服务器运行的用户）身份登录，请确保文件具有允许`mysql`读取的权限。

1.  启动 MySQL 服务器，并将`init_file`系统变量设置为命名的文件：

    ```sql
    $> mysqld --init-file=/home/me/mysql-init &
    ```

    服务器在启动时执行由`init_file`系统变量命名的文件的内容，更改`'root'@'localhost'`账户的密码。

    还可能需要其他选项，具体取决于您通常如何启动服务器。例如，在`init_file`参数之前可能需要`--defaults-file`。

1.  服务器成功启动后，删除`/home/me/mysql-init`。

现在，您应该能够使用新密码连接到 MySQL 服务器作为`root`。停止服务器，然后正常重启。

##### B.3.3.2.3 重置根密码：通用说明

前面的章节专门提供了针对 Windows 和类 Unix 系统的重置密码说明。或者，在任何平台上，您都可以使用**mysql**客户端来重置密码（但这种方法不够安全）：

1.  必要时停止 MySQL 服务器，然后使用`--skip-grant-tables`选项重新启动。这将使任何人都可以连接而无需密码，并具有所有权限，并禁用帐户管理语句，如`ALTER USER`和`SET PASSWORD`。由于这是不安全的，如果服务器使用`--skip-grant-tables`选项启动，它还通过启用`skip_networking`来禁用远程连接。

1.  使用**mysql**客户端连接到 MySQL 服务器；不需要密码，因为服务器是使用`--skip-grant-tables`启动的：

    ```sql
    $> mysql
    ```

1.  在`mysql`客户端中，告诉服务器重新加载授权表，以便帐户管理语句起作用：

    ```sql
    mysql> FLUSH PRIVILEGES;
    ```

    然后更改`'root'@'localhost'`账户的密码。将密码替换为您想要使用的密码。要更改具有不同主机名部分的`root`账户的密码，请修改指令以使用该主机名。

    ```sql
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
    ```

现在，您应该能够使用新密码作为`root`连接到 MySQL 服务器。停止服务器并正常重启（不使用`--skip-grant-tables`选项，也不启用`skip_networking`系统变量）。
