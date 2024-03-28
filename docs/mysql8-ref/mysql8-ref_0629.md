> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-symbolic-links.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-symbolic-links.html)

#### 10.12.2.3 在 Windows 上使用符号链接进行数据库操作

在 Windows 上，可以使用符号链接来创建数据库目录。这使您可以通过设置符号链接将数据库目录放在不同的位置（例如，不同的磁盘）上。在 Windows 上使用数据库符号链接类似于在 Unix 上的使用，尽管设置链接的过程有所不同。

假设您想要将名为 `mydb` 的数据库的数据库目录放在 `D:\data\mydb`。为此，在 MySQL 数据目录中创建一个指向 `D:\data\mydb` 的符号链接。但是，在创建符号链接之前，请确保 `D:\data\mydb` 目录存在，必要时进行创建。如果您已经在数据目录中有一个名为 `mydb` 的数据库目录，请将其移动到 `D:\data`。否则，符号链接将不起作用。为避免问题，请确保在移动数据库目录时服务器未运行。

在 Windows 上，您可以使用 **mklink** 命令创建符号链接。此命令需要管理员权限。

1.  确保所需的数据库路径存在。在本示例中，我们使用 `D:\data\mydb` 和一个名为 `mydb` 的数据库。

1.  如果数据库尚不存在，请在 **mysql** 客户端中发出 `CREATE DATABASE mydb` 来创建它。

1.  停止 MySQL 服务。

1.  使用 Windows 资源管理器或命令行，将数据目录中的 `mydb` 目录移动到 `D:\data`，替换同名目录。

1.  如果您尚未使用命令提示符，请打开它，并将位置更改为数据目录，如下所示：

    ```sql
    C:\> cd *\path\to\datadir*
    ```

    如果您的 MySQL 安装在默认位置，您可以使用以下命令：

    ```sql
    C:\> cd C:\ProgramData\MySQL\MySQL Server 8.0\Data
    ```

1.  在数据目录中，创建一个名为 `mydb` 的符号链接，指向数据库目录的位置：

    ```sql
    C:\> mklink /d mydb D:\data\mydb
    ```

1.  启动 MySQL 服务。

之后，所有在数据库 `mydb` 中创建的表都将在 `D:\data\mydb` 中创建。

或者，在 MySQL 支持的任何 Windows 版本上，您可以通过在数据目录中创建一个包含指向目标目录路径的 `.sym` 文件来创建 MySQL 数据库的符号链接。该文件应命名为 `*`db_name`*.sym`，其中 *`db_name`* 是数据库名称。

Windows 上默认启用使用 `.sym` 文件创建数据库符号链接的支持。如果您不需要 `.sym` 文件符号链接，可以通过使用 `--skip-symbolic-links` 选项启动 **mysqld** 来禁用对它们的支持。要确定您的系统是否支持 `.sym` 文件符号链接，请使用以下语句检查 `have_symlink` 系统变量的值：

```sql
SHOW VARIABLES LIKE 'have_symlink';
```

要创建 `.sym` 文件符号链接，请按照以下步骤进行：

1.  将位置更改为数据目录：

    ```sql
    C:\> cd *\path\to\datadir*
    ```

1.  在数据目录中，创建一个名为`mydb.sym`的文本文件，其中包含此路径名：`D:\data\mydb\`

    注意

    新数据库和表的路径名应为绝对路径。如果您指定相对路径，则位置相对于`mydb.sym`文件。

在此之后，所有在数据库`mydb`中创建的表都将被创建在`D:\data\mydb`中。
