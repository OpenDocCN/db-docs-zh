> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-windows-command-line-servers.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-windows-command-line-servers.html)

#### 7.8.2.1 在 Windows 命令行上启动多个 MySQL 实例

从命令行手动启动单个 MySQL 服务器的过程在 Section 2.3.4.6, “Starting MySQL from the Windows Command Line”中有描述。要以这种方式启动多个服务器，可以在命令行或选项文件中指定适当的选项。将选项放在选项文件中更方便，但必须确保每个服务器都有自己的选项集。为此，请为每个服务器创建一个选项文件，并在运行时使用`--defaults-file`选项告诉服务器文件名。

假设您想要在端口 3307 上运行一个**mysqld**实例，数据目录为`C:\mydata1`，另一个实例在端口 3308 上，数据目录为`C:\mydata2`。使用以下步骤：

1.  确保每个数据目录都存在，包括包含授权表的`mysql`数据库的副本。

1.  创建两个选项文件。例如，创建一个名为`C:\my-opts1.cnf`的文件，内容如下：

    ```sql
    [mysqld]
    datadir = C:/mydata1
    port = 3307
    ```

    创建第二个名为`C:\my-opts2.cnf`的文件，内容如下：

    ```sql
    [mysqld]
    datadir = C:/mydata2
    port = 3308
    ```

1.  使用`--defaults-file`选项启动每个服务器时都使用自己的选项文件：

    ```sql
    C:\> C:\mysql\bin\mysqld --defaults-file=C:\my-opts1.cnf
    C:\> C:\mysql\bin\mysqld --defaults-file=C:\my-opts2.cnf
    ```

    每个服务器都在前台运行（直到服务器稍后退出时不会出现新提示），因此您需要在单独的控制台窗口中发出这两个命令。

要关闭服务器，请使用适当的端口号连接到每个服务器：

```sql
C:\> C:\mysql\bin\mysqladmin --port=3307 --host=127.0.0.1 --user=root --password shutdown
C:\> C:\mysql\bin\mysqladmin --port=3308 --host=127.0.0.1 --user=root --password shutdown
```

如上所述配置的服务器允许客户端通过 TCP/IP 连接。如果您的 Windows 版本支持命名管道，并且您还希望允许命名管道连接，请指定启用命名管道的选项并指定其名称。支持命名管道连接的每个服务器必须使用唯一的管道名称。例如，`C:\my-opts1.cnf`文件可能如下所示：

```sql
[mysqld]
datadir = C:/mydata1
port = 3307
enable-named-pipe
socket = mypipe1
```

类似地修改`C:\my-opts2.cnf`以供第二个服务器使用。然后按照先前描述的方式启动服务器。

对于希望允许共享内存连接的服务器，也适用类似的过程。通过启动具有启用`shared_memory`系统变量的服务器，并通过设置`shared_memory_base_name`系统变量来为每个服务器指定唯一的共享内存名称，启用此类连接。
