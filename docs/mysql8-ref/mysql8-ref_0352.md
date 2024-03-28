> 原文：[`dev.mysql.com/doc/refman/8.0/en/password-security-user.html`](https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html)

#### 8.1.2.1 密码安全的最终用户准则

MySQL 用户应该遵循以下准则来保护密码安全。

当您运行客户端程序连接到 MySQL 服务器时，不建议以暴露给其他用户发现的方式指定密码。您运行客户端程序时可以使用的指定密码的方法在此列出，以及每种方法的风险评估。简而言之，最安全的方法是让客户端程序提示输入密码或在正确保护的选项文件中指定密码。

+   使用**mysql_config_editor**实用程序，它允许您将身份验证凭据存储在名为`.mylogin.cnf`的加密登录路径文件中。稍后，MySQL 客户端程序可以读取该文件以获取连接到 MySQL 服务器的身份验证凭据。请参阅 Section 6.6.7, “mysql_config_editor — MySQL Configuration Utility”。

+   在命令行上使用`--password=*`password`*`或`-p*`password`*`选项。例如：

    ```sql
    $> mysql -u francis -pfrank *db_name*
    ```

    警告

    这很方便*但不安全*。在某些系统上，您的密码会对系统状态程序（如**ps**）可见，其他用户可以调用这些程序来显示命令行。MySQL 客户端通常在初始化序列期间用零覆盖命令行密码参数。然而，在这个值可见的瞬间仍然存在。此外，在某些系统上，这种覆盖策略是无效的，密码仍然对**ps**可见。（SystemV Unix 系统和其他系统可能存在这个问题。）

    如果您的操作环境设置为在终端窗口的标题栏中显示当前命令，则只要命令正在运行，密码就会保持可见，即使命令已经在窗口内容区域滚动出视野。

+   在命令行上使用`--password`或`-p`选项，不指定密码值。在这种情况下，客户端程序会交互式地请求密码：

    ```sql
    $> mysql -u francis -p *db_name*
    Enter password: ********
    ```

    `*`字符表示您输入密码的位置。在输入时不会显示密码。

    以这种方式输入密码比在命令行上指定密码更安全，因为其他用户看不到密码。然而，这种输入密码的方法只适用于您交互式运行的程序。如果您想从非交互式脚本中调用客户端，那么就没有机会从键盘输入密码。在某些系统上，您甚至可能发现脚本的第一行被读取并解释（错误地）为您的密码。

+   将密码存储在一个选项文件中。例如，在 Unix 上，您可以在主目录的`.my.cnf`文件的`[client]`部分列出您的密码：

    ```sql
    [client]
    password=*password*
    ```

    为了保护密码安全，该文件不应该对任何人开放访问，只能由您自己访问。为了确保这一点，将文件访问模式设置为`400`或`600`。例如：

    ```sql
    $> chmod 600 .my.cnf
    ```

    从命令行命名包含密码的特定选项文件，使用`--defaults-file=*`file_name`*`选项，其中`file_name`是文件的完整路径名。例如：

    ```sql
    $> mysql --defaults-file=/home/francis/mysql-opts
    ```

    Section 6.2.2.2, “Using Option Files”详细讨论了选项文件。

在 Unix 系统上，**mysql**客户端会将执行的语句记录到一个历史文件中（参见 Section 6.5.1.3, “mysql Client Logging”）。默认情况下，该文件名为`.mysql_history`，并创建在您的主目录中。密码可以以明文形式写入 SQL 语句，比如`CREATE USER`和`ALTER USER`，因此如果您使用这些语句，它们会被记录在历史文件中。为了保护这个文件，使用限制性的访问模式，与之前描述的`.my.cnf`文件相同。

如果您的命令解释器维护一个历史记录，保存命令的任何文件都包含在命令行中输入的 MySQL 密码。例如，**bash**使用`~/.bash_history`。任何这样的文件都应该具有限制性的访问模式。
