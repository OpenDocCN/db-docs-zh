> 原文：[`dev.mysql.com/doc/refman/8.0/en/multiple-windows-services.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-windows-services.html)

#### 7.8.2.2 启动多个 MySQL 实例作为 Windows 服务

在 Windows 上，MySQL 服务器可以作为 Windows 服务运行。有关安装、控制和删除单个 MySQL 服务的过程，请参阅 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

要设置多个 MySQL 服务，您必须确保每个实例除了其他必须针对每个实例唯一的参数外，还使用不同的服务名称。

对于以下说明，假设您想从分别安装在`C:\mysql-5.7.9`和`C:\mysql-8.0.36`的两个不同版本的 MySQL 运行**mysqld**服务器。（如果您正在运行 5.7.9 作为生产服务器，但也想使用 8.0.36 进行测试，可能会出现这种情况。）

要将 MySQL 安装为 Windows 服务，请使用`--install`或`--install-manual`选项。有关这些选项的信息，请参阅 Section 2.3.4.8, “Starting MySQL as a Windows Service”。

根据前面的信息，您有几种设置多个服务的方法。以下说明描述了一些示例。在尝试任何方法之前，请关闭并删除任何现有的 MySQL 服务。

+   **方法 1：** 在一个标准选项文件中为所有服务指定选项。为此，为每个服务器使用不同的服务名称。假设您想要使用服务名称`mysqld1`运行 5.7.9 **mysqld**，并使用服务名称`mysqld2`运行 8.0.36 **mysqld**。在这种情况下，您可以为 5.7.9 使用`[mysqld1]`组，为 8.0.36 使用`[mysqld2]`组。例如，您可以像这样设置`C:\my.cnf`：

    ```sql
    # options for mysqld1 service
    [mysqld1]
    basedir = C:/mysql-5.7.9
    port = 3307
    enable-named-pipe
    socket = mypipe1

    # options for mysqld2 service
    [mysqld2]
    basedir = C:/mysql-8.0.36
    port = 3308
    enable-named-pipe
    socket = mypipe2
    ```

    安装服务如下，使用完整的服务器路径名确保 Windows 为每个服务注册正确的可执行程序：

    ```sql
    C:\> C:\mysql-5.7.9\bin\mysqld --install mysqld1
    C:\> C:\mysql-8.0.36\bin\mysqld --install mysqld2
    ```

    要启动服务，请使用服务管理器，或使用适当的服务名称使用**NET START**或**SC START**：

    ```sql
    C:\> SC START mysqld1
    C:\> SC START mysqld2
    ```

    要停止服务，请使用服务管理器，或使用适当的服务名称使用**NET STOP**或**SC STOP**：

    ```sql
    C:\> SC STOP mysqld1
    C:\> SC STOP mysqld2
    ```

+   **方法 2：** 在单独的文件中为每个服务器指定选项，并在安装服务时使用`--defaults-file`告诉每个服务器要使用哪个文件。在这种情况下，每个文件应该使用`[mysqld]`组列出选项。

    使用这种方法，要为 5.7.9 **mysqld**指定选项，请创建一个看起来像这样的文件`C:\my-opts1.cnf`：

    ```sql
    [mysqld]
    basedir = C:/mysql-5.7.9
    port = 3307
    enable-named-pipe
    socket = mypipe1
    ```

    对于 8.0.36 版本的**mysqld**，创建一个名为`C:\my-opts2.cnf`的文件，内容如下：

    ```sql
    [mysqld]
    basedir = C:/mysql-8.0.36
    port = 3308
    enable-named-pipe
    socket = mypipe2
    ```

    安装服务的步骤如下（每个命令单独一行输入）：

    ```sql
    C:\> C:\mysql-5.7.9\bin\mysqld --install mysqld1
               --defaults-file=C:\my-opts1.cnf
    C:\> C:\mysql-8.0.36\bin\mysqld --install mysqld2
               --defaults-file=C:\my-opts2.cnf
    ```

    当将 MySQL 服务器安装为服务并使用`--defaults-file`选项时，服务名称必须在选项之前。

    安装服务后，启动和停止服务的方法与前面的示例相同。

要删除多个服务，请对每个服务使用**SC DELETE *`mysqld_service_name`***。或者，对每个服务使用**mysqld --remove**，在`--remove`选项后指定服务名称。如果服务名称是默认值（`MySQL`），在使用**mysqld --remove**时可以省略它。
