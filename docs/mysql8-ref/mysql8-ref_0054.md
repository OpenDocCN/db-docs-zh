> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-start-service.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-start-service.html)

#### 2.3.4.8 在 Windows 上启动 MySQL 作为服务

在 Windows 上，运行 MySQL 的推荐方式是将其安装为 Windows 服务，这样当 Windows 启动和停止时，MySQL 也会自动启动和停止。安装为服务的 MySQL 服务器也可以使用**NET**命令或图形**服务**实用程序从命令行进行控制。通常，要将 MySQL 安装为 Windows 服务，您应该使用具有管理员权限的帐户登录。

**服务**实用程序（Windows **服务控制管理器**）可以在 Windows 控制面板中找到。为了避免冲突，在执行服务器安装或从命令行执行删除操作时，建议关闭**服务**实用程序。

##### 安装服务

在将 MySQL 安装为 Windows 服务之前，如果当前服务器正在运行，您应该首先使用以下命令停止当前服务器：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqladmin"
          -u root shutdown
```

注意

如果 MySQL `root` 用户帐户有密码，您需要使用 `-p` 选项调用 **mysqladmin** 并在提示时提供密码。

此命令调用 MySQL 管理实用程序 **mysqladmin** 连接到服务器并告诉它关闭。该命令连接为 MySQL `root` 用户，这是 MySQL 授权系统中的默认管理帐户。

注意

MySQL 授权系统中的用户与 Windows 下的任何操作系统用户完全独立。

使用以下命令将服务器安装为服务：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld" --install
```

服务安装命令不会启动服务器。关于如何启动服务器的说明稍后在本节中给出。

为了更容易调用 MySQL 程序，您可以将 MySQL `bin` 目录的路径名添加到您的 Windows 系统 `PATH` 环境变量中：

+   在 Windows 桌面上，右键单击“我的电脑”图标，然后选择“属性”。

+   接下来，从出现的系统属性菜单中选择“高级”选项卡，然后点击“环境变量”按钮。

+   在“系统变量”下，选择“Path”，然后点击“编辑”按钮。应该会出现“编辑系统变量”对话框。

+   将光标放在标记为变量值的空格中显示的文本末尾。（使用**End**键确保光标位于此空格中文本的末尾。）然后输入您的 MySQL `bin`目录的完整路径名（例如，`C:\Program Files\MySQL\MySQL Server 8.0\bin`），并且此路径与此字段中存在的任何值之间应有一个分号分隔。关闭此对话框，以及依次关闭每个对话框，直到打开的所有对话框都已关闭。现在，您应该能够在系统上的任何目录中键入其名称来调用任何 MySQL 可执行程序，而无需提供路径。这包括服务器、**mysql**客户端以及所有 MySQL 命令行实用程序，如**mysqladmin**和**mysqldump**。

    如果您在同一台计算机上运行多个 MySQL 服务器，则不应将 MySQL `bin`目录添加到 Windows `PATH`中。

警告

当手动编辑系统`PATH`时，务必要非常小心；意外删除或修改现有`PATH`值的任何部分都可能导致系统发生故障，甚至无法使用。

安装服务时可以使用以下附加参数：

+   您可以在`--install`选项后立即指定服务名称。默认服务名称是`MySQL`。

+   如果给出了服务名称，则可以跟随一个选项。按照惯例，这应该是`--defaults-file=*`file_name`*`，以指定服务器启动时应从中读取选项的选项文件的名称。

    可以使用除`--defaults-file`之外的单个选项，但不建议这样做。`--defaults-file`更灵活，因为它允许您通过将它们放在命名的选项文件中来指定服务器的多个启动选项。

+   您还可以在服务名称后面指定`--local-service`选项。这将导致服务器使用具有有限系统特权的`LocalService` Windows 帐户运行。如果在服务名称后面给出了`--defaults-file`和`--local-service`，它们可以以任何顺序出现。

对于作为 Windows 服务安装的 MySQL 服务器，以下规则确定服务器使用的服务名称和选项文件：

+   如果服务安装命令未指定服务名称或在`--install`选项后跟随默认服务名称（`MySQL`），服务器将使用`MySQL`的服务名称，并从标准选项文件中的`[mysqld]`组中读取选项。

+   如果服务安装命令在 `--install` 选项后指定了除 `MySQL` 之外的服务名称，则服务器将使用该服务名称。它从标准选项文件中的 `[mysqld]` 组和与服务名称相同的组中读取选项。这使您可以使用 `[mysqld]` 组来设置所有 MySQL 服务都应使用的选项，并使用具有服务名称的选项组来为使用该服务名称安装的服务器使用选项。

+   如果服务安装命令在服务名称之后指定了 `--defaults-file` 选项，则服务器将按照前一项描述的方式读取选项，只是它只从命名文件中读取选项，而忽略标准选项文件。

作为更复杂的示例，请考虑以下命令：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld"
          --install MySQL --defaults-file=C:\my-opts.cnf
```

在这里，`--install` 选项后给出了默认服务名称（`MySQL`）。如果没有给出 `--defaults-file` 选项，此命令将导致服务器从标准选项文件中读取 `[mysqld]` 组。然而，由于存在 `--defaults-file` 选项，服务器将从 `[mysqld]` 选项组中读取选项，而且只从命名文件中读取。

注意

在 Windows 上，如果使用 `--defaults-file` 和 `--install` 选项启动服务器，则必须首先使用 `--install`。否则，`mysqld.exe` 将尝试启动 MySQL 服务器。

您还可以在启动 MySQL 服务之前在 Windows **服务** 实用程序中将选项指定为启动参数。

最后，在尝试启动 MySQL 服务之前，请确保操作系统用户的用户变量 `%TEMP%` 和 `%TMP%`（以及 `%TMPDIR%`，如果曾经设置过）指向用户具有写入权限的文件夹。运行 MySQL 服务的默认用户是 `LocalSystem`，其 `%TEMP%` 和 `%TMP%` 的默认值是 `C:\Windows\Temp`，`LocalSystem` 默认具有写入权限。然而，如果对默认设置进行了任何更改（例如，更改运行服务的用户或提到的用户变量，或使用 `--tmpdir` 选项将临时目录放在其他位置），MySQL 服务可能无法运行，因为未授予适当用户对临时目录的写入权限。

##### 启动服务

安装为服务后，每当 Windows 启动时，Windows 会自动启动 MySQL 服务器实例。也可以立即从**Services**实用程序启动服务，或使用**sc start *`mysqld_service_name`***或**NET START *`mysqld_service_name`***命令启动服务。**SC**和**NET**命令不区分大小写。

当作为服务运行时，**mysqld**无法访问控制台窗口，因此无法在那里看到任何消息。如果**mysqld**无法启动，请检查错误日志，查看服务器是否在那里写入任何消息以指示问题的原因。错误日志位于 MySQL 数据目录中（例如，`C:\Program Files\MySQL\MySQL Server 8.0\data`）。它是带有后缀`.err`的文件。

当 MySQL 服务器安装为服务且服务正在运行时，Windows 在关闭 Windows 时会自动停止服务。也可以使用`Services`实用程序、**sc stop *`mysqld_service_name`***命令、**NET STOP *`mysqld_service_name`***命令或**mysqladmin shutdown**命令手动停止服务器。

如果您不希望服务在启动过程中自动启动，还可以选择将服务器安装为手动服务。要做到这一点，请使用`--install-manual`选项而不是`--install`选项：

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld" --install-manual
```

##### 移除服务

要删除安装为服务的服务器，首先停止正在运行的服务器，执行**SC STOP *`mysqld_service_name`***或**NET STOP *`mysqld_service_name`***。然后使用**SC DELETE *`mysqld_service_name`***来删除它：

```sql
C:\> SC DELETE mysql
```

或者，使用**mysqld**`--remove`选项来移除服务。

```sql
C:\> "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqld" --remove
```

如果**mysqld**未作为服务运行，则可以从命令行启动它。有关说明，请参阅第 2.3.4.6 节，“从 Windows 命令行启动 MySQL”。

如果在安装过程中遇到困难，请参阅第 2.3.5 节，“解决 Microsoft Windows MySQL 服务器安装问题”。

有关停止或移除 Windows 服务的更多信息，请参阅第 7.8.2.2 节，“将多个 MySQL 实例作为 Windows 服务启动”。
