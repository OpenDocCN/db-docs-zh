# 2.3.5 解决 Microsoft Windows MySQL 服务器安装问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-troubleshooting.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-troubleshooting.html)

第一次安装和运行 MySQL 时，您可能会遇到一些错误，导致 MySQL 服务器无法启动。本节帮助您诊断和纠正其中一些错误。

在解决服务器问题时，您的第一个资源是错误日志。MySQL 服务器使用错误日志记录与阻止服务器启动的错误相关的信息。错误日志位于您的`my.ini`文件中指定的数据目录中。默认数据目录位置为`C:\Program Files\MySQL\MySQL Server 8.0\data`，或者在 Windows 7 和 Windows Server 2008 上为`C:\ProgramData\Mysql`。`C:\ProgramData`目录默认为隐藏。您需要更改文件夹选项才能查看目录和内容。有关错误日志和内容理解的更多信息，请参阅 Section 7.4.2, “The Error Log”。

有关可能错误的信息，还请查看在启动 MySQL 服务时显示的控制台消息。在将**mysqld**安装为服务后，使用命令行中的**SC START *`mysqld_service_name`***或**NET START *`mysqld_service_name`***命令查看有关启动 MySQL 服务器作为服务时的任何错误消息。请参阅 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

以下示例显示您在首次安装 MySQL 并启动服务器时可能遇到的其他常见错误消息：

+   如果 MySQL 服务器无法找到`mysql`权限数据库或其他关键文件，则会显示以下消息：

    ```sql
    System error 1067 has occurred.
    Fatal error: Can't open and lock privilege tables:
    Table 'mysql.user' doesn't exist
    ```

    当 MySQL 基本或数据目录安装在与默认位置（分别为`C:\Program Files\MySQL\MySQL Server 8.0`和`C:\Program Files\MySQL\MySQL Server 8.0\data`）不同的位置时，通常会出现���些消息。

    当 MySQL 升级并安装到新位置时，可能会出现这种情况，但配置文件没有更新以反映新位置。此外，旧配置文件和新配置文件可能会发生冲突。在升级 MySQL 时，请务必删除或重命名任何旧配置文件。

    如果您将 MySQL 安装到除`C:\Program Files\MySQL\MySQL Server 8.0`之外的目录，请通过使用配置（`my.ini`）文件确保 MySQL 服务器知道这一点。将`my.ini`文件放在您的 Windows 目录中，通常为`C:\WINDOWS`。要根据`WINDIR`环境变量的值确定其确切位置，请从命令提示符中发出以下命令：

    ```sql
    C:\> echo %WINDIR%
    ```

    你可以使用任何文本编辑器（如记事本）创建或修改选项文件。例如，如果 MySQL 安装在 `E:\mysql`，数据目录是 `D:\MySQLdata`，你可以创建选项文件并设置 `[mysqld]` 部分以指定 `basedir` 和 `datadir` 选项的值：

    ```sql
    [mysqld]
    # set basedir to your installation path
    basedir=E:/mysql
    # set datadir to the location of your data directory
    datadir=D:/MySQLdata
    ```

    Microsoft Windows 路径名在选项文件中使用（正斜杠）而不是反斜杠。如果使用反斜杠，请将其双写：

    ```sql
    [mysqld]
    # set basedir to your installation path
    basedir=C:\\Program Files\\MySQL\\MySQL Server 8.0
    # set datadir to the location of your data directory
    datadir=D:\\MySQLdata
    ```

    在选项文件值中使用反斜杠的规则详见 第 6.2.2.2 节，“使用选项文件”。

    如果在 MySQL 配置文件中更改了 `datadir` 值，则必须在重新启动 MySQL 服务器之前移动现有 MySQL 数据目录中的内容。

    参见 第 2.3.4.2 节，“创建选项文件”。

+   如果在先停止和删除现有 MySQL 服务的情况下重新安装或升级 MySQL，并使用 MySQL 安装程序安装 MySQL，则可能会看到此错误：

    ```sql
    Error: Cannot create Windows service for MySql. Error: 0
    ```

    当配置向导尝试安装服务并发现同名现有服务时会发生这种情况。

    解决此问题的一个方法是在使用配置向导时选择一个不同于 `mysql` 的服务名称。这样可以正确安装新服务，但会保留过时的服务。虽然这是无害的，最好是删除不再使用的旧服务。

    要永久删除旧的 `mysql` 服务，请以具有管理员权限的用户在命令行上执行以下命令：

    ```sql
    C:\> SC DELETE mysql
    [SC] DeleteService SUCCESS
    ```

    如果你的 Windows 版本没有 `SC` 实用程序，请从 [`www.microsoft.com/windows2000/techinfo/reskit/tools/existing/delsrv-o.asp`](http://www.microsoft.com/windows2000/techinfo/reskit/tools/existing/delsrv-o.asp) 下载 `delsrv` 实用程序，并使用 `delsrv mysql` 语法。
