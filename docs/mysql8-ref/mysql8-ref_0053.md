> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-installation-windows-path.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-installation-windows-path.html)

#### 2.3.4.7 自定义 MySQL 工具的 PATH

警告

当手动编辑系统`PATH`时，您必须非常小心；意外删除或修改现有`PATH`值的任何部分可能导致系统发生故障，甚至无法使用。

为了更容易调用 MySQL 程序，您可以将 MySQL `bin` 目录的路径名添加到您的 Windows 系统 `PATH` 环境变量中：

+   在 Windows 桌面上，右键单击“我的电脑”图标，然后选择“属性”。

+   接下来，从出现的“系统属性”菜单中选择“高级”选项卡，然后单击“环境变量”按钮。

+   在“系统变量”下，选择“Path”，然后单击“编辑”按钮。应该会出现“编辑系统变量”对话框。

+   将光标放在标有“变量值”的空格中显示的文本末尾。（使用 **End** 键确保光标位于此空格中文本的最末尾。）然后输入您的 MySQL `bin` 目录的完整路径名（例如，`C:\Program Files\MySQL\MySQL Server 8.0\bin`）

    注意

    在此路径与该字段中的任何值之间必须有一个分号分隔。

    通过单击“确定”来关闭此对话框，然后依次关闭每个对话框，直到所有打开的对话框都被关闭。新的`PATH`值现在应该对您打开的任何新命令 shell 可用，允许您在系统上的任何目录中键入其名称以在 DOS 提示符下调用任何 MySQL 可执行程序，而无需提供路径。这包括服务器、**mysql** 客户端以及所有 MySQL 命令行实用程序，如 **mysqladmin** 和 **mysqldump**。

如果您在同一台机器上运行多个 MySQL 服务器，则不应将 MySQL `bin` 目录添加到您的 Windows `PATH` 中。
