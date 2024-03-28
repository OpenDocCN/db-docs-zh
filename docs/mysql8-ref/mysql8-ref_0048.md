> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-create-option-file.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-create-option-file.html)

#### 2.3.4.2 创建一个选项文件

如果在运行服务器时需要指定启动选项，可以在命令行中指定它们或将它们放在选项文件中。对于每次服务器启动都使用的选项，最方便的方法可能是使用选项文件来指定 MySQL 配置。特别是在以下情况下：

+   安装或数据目录位置与默认位置（`C:\Program Files\MySQL\MySQL Server 8.0` 和 `C:\Program Files\MySQL\MySQL Server 8.0\data`）不同。

+   你需要调整服务器设置，比如内存、缓存或 InnoDB 配置信息。

当 MySQL 服务器在 Windows 上启动时，它会在几个位置查找选项文件，比如 Windows 目录、`C:\` 和 MySQL 安装目录（有关所有位置的完整列表，请参见第 6.2.2.2 节，“使用选项文件”）。Windows 目录通常被命名为类似`C:\WINDOWS`的东西。您可以通过以下命令使用`WINDIR`环境变量的值确定其确切位置：

```sql
C:\> echo %WINDIR%
```

MySQL 首先在每个位置的`my.ini`文件中查找选项，然后在`my.cnf`文件中查找。然而，为避免混淆，最好只使用一个文件。如果您的 PC 使用一个引导加载程序，其中`C:`不是引导驱动器，您唯一的选择是使用`my.ini`文件。无论使用哪个选项文件，它必须是一个纯文本文件。

注意

在使用 MySQL Installer 安装 MySQL 服务器时，它会在默认位置创建`my.ini`，并且执行 MySQL Installer 的用户被授予对这个新的`my.ini`文件的完全权限。

换句话说，确保 MySQL 服务器用户有权限读取`my.ini`文件。

你也可以利用随 MySQL 发行版附带的示例选项文件；参见第 7.1.2 节，“服务器配置默认值”。

选项文件可以使用任何文本编辑器创建和修改，比如记事本。例如，如果 MySQL 安装在`E:\mysql`，数据目录在`E:\mydata\data`，你可以创建一个包含`[mysqld]`部分以指定`basedir`和`datadir`选项值的选项文件：

```sql
[mysqld]
# set basedir to your installation path
basedir=E:/mysql
# set datadir to the location of your data directory
datadir=E:/mydata/data
```

Microsoft Windows 路径名在选项文件中使用（正斜杠）而不是反斜杠。如果使用反斜杠，需要双写：

```sql
[mysqld]
# set basedir to your installation path
basedir=E:\\mysql
# set datadir to the location of your data directory
datadir=E:\\mydata\\data
```

有关选项文件值中反斜杠使用规则，请参见第 6.2.2.2 节，“使用选项文件”。

ZIP 归档文件不包括`data`目录。要通过创建数据目录并填充 mysql 系统数据库中的表来初始化 MySQL 安装，请使用`--initialize`或`--initialize-insecure`。有关更多信息，请参见 Section 2.9.1，“初始化数据目录”。

如果您想要在不同位置使用数据目录，您应该将`data`目录的所有内容复制到新位置。例如，如果您想要将`E:\mydata`作为数据目录，您必须执行两项操作：

1.  将整个`data`目录及其所有内容从默认位置（例如`C:\Program Files\MySQL\MySQL Server 8.0\data`）移动到`E:\mydata`。

1.  每次启动服务器时使用`--datadir`选项指定新数据目录位置。
