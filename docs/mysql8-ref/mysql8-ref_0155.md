> 原文：[`dev.mysql.com/doc/refman/8.0/en/option-files.html`](https://dev.mysql.com/doc/refman/8.0/en/option-files.html)

#### 6.2.2.2 使用选项文件

大多数 MySQL 程序可以从选项文件（有时称为配置文件）中读取启动选项。选项文件提供了一种方便的方式来指定常用选项，这样每次运行程序时就不需要在命令行中输入它们。

要确定一个程序是否读取选项文件，请使用`--help`选项调用它。（对于**mysqld**，使用`--verbose`和`--help`。）如果程序读取选项文件，帮助消息会指示它查找哪些文件以及识别哪些选项组。

注意

使用`--no-defaults`选项启动的 MySQL 程序不会读取除`.mylogin.cnf`之外的任何选项文件。

禁用`persisted_globals_load`系统变量启动的服务器不会读取`mysqld-auto.cnf`。

许多选项文件都是纯文本文件，可以使用任何文本编辑器创建。例外情况包括：

+   包含登录路径选项的`.mylogin.cnf`文件。这是由**mysql_config_editor**实用程序创建的加密文件。参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。“登录路径”是一个只允许特定选项的选项组：`host`、`user`、`password`、`port`和`socket`。客户端程序使用`--login-path`选项指定从`.mylogin.cnf`中读取哪个登录路径。

    要指定替代的登录路径文件名，请设置`MYSQL_TEST_LOGIN_FILE`环境变量。这个变量被**mysql-test-run.pl**测试实用程序使用，但也被**mysql_config_editor**和 MySQL 客户端（如**mysql**、**mysqladmin**等）所识别。

+   数据目录中的`mysqld-auto.cnf`文件。这个 JSON 格式的文件包含持久化的系统变量设置。它是由服务器在执行`SET PERSIST`或`SET PERSIST_ONLY`语句时创建的。参见第 7.1.9.3 节，“持久化系统变量”。`mysqld-auto.cnf`的管理应该交给服务器处理，不要手动执行。

+   选项文件处理顺序

+   选项文件语法

+   选项文件包含

##### 选项文件处理顺序

MySQL 按照以下讨论中描述的顺序查找选项文件并读取存在的任何选项文件。如果要使用的选项文件不存在，请使用适当的方法创建，如前文所述。

注意

有关与 NDB Cluster 程序一起使用的选项文件的信息，请参见第 25.4 节，“NDB Cluster 的配置”。

在 Windows 上，MySQL 程序按照以下表格中显示的顺序读取启动选项（先读取列出的文件，后读取的文件优先）。

**表 6.1 Windows 系统上读取的选项文件**

| 文件名 | 目的 |
| --- | --- |
| ``%WINDIR%`\my.ini`, ``%WINDIR%`\my.cnf` | 全局选项 |
| `C:\my.ini`, `C:\my.cnf` | 全局选项 |
| `*`BASEDIR`*\my.ini`, `*`BASEDIR`*\my.cnf` | 全局选项 |
| `defaults-extra-file` | 使用`--defaults-extra-file`指定的文件（如果有） |
| ``%APPDATA%`\MySQL\.mylogin.cnf` | 登录路径选项（仅限客户端） |
| `*`DATADIR`*\mysqld-auto.cnf` | 使用`SET PERSIST`或`SET PERSIST_ONLY`持久化的系统变量（仅限服务器） |

在上表中，`%WINDIR%`代表 Windows 目录的位置。通常为`C:\WINDOWS`。使用以下命令根据`WINDIR`环境变量的值确定其确切位置：

```sql
C:\> echo %WINDIR%
```

`%APPDATA%`代表 Windows 应用程序数据目录的值。使用以下命令根据`APPDATA`环境变量的值确定其确切位置：

```sql
C:\> echo %APPDATA%
```

*`BASEDIR`*代表 MySQL 基本安装目录。当使用 MySQL Installer 安装 MySQL 8.0 时，这通常是`C:\*`PROGRAMDIR`*\MySQL\MySQL Server 8.0`，其中*`PROGRAMDIR`*代表程序目录（通常为英语版 Windows 的`Program Files`）。参见第 2.3.3 节，“Windows 上的 MySQL Installer”。

重要提示

尽管 MySQL Installer 将大多数文件放在*`PROGRAMDIR`*下，但默认情况下会将`my.ini`安装在`C:\ProgramData\MySQL\MySQL Server 8.0\`目录下。

*`DATADIR`*代表 MySQL 数据目录。作为查找`mysqld-auto.cnf`的默认值是 MySQL 编译时内置的数据目录位置，但可以通过在处理`mysqld-auto.cnf`之前处理的选项文件或命令行选项`--datadir`进行更改。

在 Unix 和类 Unix 系统上，MySQL 程序按照以下表中显示的顺序从文件中读取启动选项（先列出的文件先读取，后读取的文件优先）。

注意

在 Unix 平台上，MySQL 会忽略全局可写的配置文件。这是一种有意为之的安全措施。

**表 6.2 Unix 和类 Unix 系统上读取的选项文件**

| 文件名 | 目的 |
| --- | --- |
| `/etc/my.cnf` | 全局选项 |
| `/etc/mysql/my.cnf` | 全局选项 |
| `*`SYSCONFDIR`*/my.cnf` | 全局选项 |
| `$MYSQL_HOME/my.cnf` | 服务器特定选项（仅服务器） |
| `defaults-extra-file` | 通过`--defaults-extra-file`指定的文件（如果有） |
| `~/.my.cnf` | 用户特定选项 |
| `~/.mylogin.cnf` | 用户特定登录路径选项（仅客户端） |
| `*`DATADIR`*/mysqld-auto.cnf` | 使用`SET PERSIST`或`SET PERSIST_ONLY`（仅服务器）持久化的系统变量 |

在上表中，`~`代表当前用户的主目录（即`$HOME`的值）。

*`SYSCONFDIR`*代表 MySQL 构建时使用的`SYSCONFDIR`选项指定的目录。默认情况下，这是编译安装目录下的`etc`目录。

`MYSQL_HOME`是一个包含服务器特定`my.cnf`文件所在目录路径的环境变量。如果未设置`MYSQL_HOME`并且使用**mysqld_safe**程序启动服务器，**mysqld_safe**会将其设置为*`BASEDIR`*，即 MySQL 基本安装目录。

*`DATADIR`*代表 MySQL 数据目录。用于查找`mysqld-auto.cnf`，其默认值是 MySQL 编译时内置的数据目录位置，但可以通过`--datadir`指定为在`mysqld-auto.cnf`处理之前处理的选项文件或命令行选项来更改。

如果找到给定选项的多个实例，则最后一个实例优先，但有一个例外：对于**mysqld**，`--user`选项的*第一个*实例作为安全预防措施，防止在命令行中覆盖选项���件中指定的用户。

##### 选项文件语法

下面对选项文件语法的描述适用于您手动编辑的文件。这不包括使用**mysql_config_editor**创建的加密的`.mylogin.cnf`文件，以及服务器以 JSON 格式创建的`mysqld-auto.cnf`文件。

运行 MySQL 程序时可以在命令行上给出的任何长选项也可以在选项文件中给出。要获取程序的可用选项列表，请使用`--help`选项运行它。（对于**mysqld**，使用`--verbose`和`--help`。）

在选项文件中指定选项的语法类似于命令行语法（参见第 6.2.2.1 节，“在命令行上使用选项”）。但是，在选项文件中，您需要省略选项名称前面的两个破折号，并且每行只指定一个选项。例如，在命令行上的`--quick`和`--host=localhost`应在选项文件中分别指定为`quick`和`host=localhost`。要在选项文件中指定形式为`--loose-*`opt_name`*`的选项，将其写为`loose-*`opt_name`*`。

选项文件中的空行将被忽略。非空行可以采用以下任一形式：

+   `#*`comment`*`，`;*`comment`*`

    注释行以`#`或`;`开头。`#`注释也可以在行中间开始。

+   `[*`group`*]`

    *`group`*是您想要设置选项的程序或组的名称。在组行之后，任何设置选项的行都适用于命名组，直到选项文件结束或另一个组行出现。选项组名称不区分大小写。

+   `*`opt_name`*`

    这等同于在命令行上的`--*`opt_name`*`。

+   `*`opt_name`*=*`value`*`

    这等同于在命令行上的`--*`opt_name`*=*`value`*`。在选项文件中，您可以在`=`字符周围有空格，这在命令行上是不成立的。值可以选择用单引号或双引号括起，如果值包含`#`注释字符，则这样做很有用。

选项名称和值的前导和尾随空格将自动删除。

您可以在选项值中使用转义序列`\b`、`\t`、`\n`、`\r`、`\\`和`\s`来表示退格、制表符、换行符、回车符、反斜杠和空格字符。在选项文件中，这些转义规则适用：

+   跟随有效转义序列字符的反斜杠将转换为序列表示的字符。例如，`\s`转换为空格。

+   未跟随有效转义序列字符的反斜杠保持不变。例如，`\S`保持不变。

前述规则意味着可以将字面反斜杠表示为`\\`，或者如果后面没有有效的转义序列字符，则表示为`\`。

选项文件中转义序列的规则与 SQL 语句中字符串文字中的转义序列的规则略有不同。在后一种情况下，如果“*`x`*”不是有效的转义序列字符，则 `\*`x`*` 变为“*`x`*”而不是 `\*`x`*`。参见 第 11.1.1 节，“字符串文字”。

选项文件值的转义规则对于使用 `\` 作为路径名分隔符的 Windows 路径名尤为重要。如果 Windows 路径名中的分隔符后面跟着一个转义序列字符，则必须将其写为 `\\`。如果没有，则可以写为 `\\` 或 `\`。另外，在 Windows 路径名中也可以使用 `/`，并且被视为 `\`。假设你想在选项文件中指定一个基本目录为 `C:\Program Files\MySQL\MySQL Server 8.0`，有几种方法可以实现。以下是一些示例：

```sql
basedir="C:\Program Files\MySQL\MySQL Server 8.0"
basedir="C:\\Program Files\\MySQL\\MySQL Server 8.0"
basedir="C:/Program Files/MySQL/MySQL Server 8.0"
basedir=C:\\Program\sFiles\\MySQL\\MySQL\sServer\s8.0
```

如果选项组名称与程序名称相同，则该组中的选项专门适用于该程序。例如，`[mysqld]` 和 `[mysql]` 组分别适用于 **mysqld** 服务器和 **mysql** 客户端程序。

`[client]` 选项组被 MySQL 发行版中提供的所有客户端程序读取（但不被 **mysqld** 读取）。要了解使用 C API 的第三方客户端程序如何使用选项文件，请参阅 mysql_options() 中的 C API 文档。

`[client]` 组使您能够指定适用于所有客户端的选项。例如，`[client]` 是指定连接到服务器的密码的适当组。 （但请确保选项文件只能被您自己访问，以防其他人发现您的密码。）确保不要将选项放在 `[client]` 组中，除非所有您使用的客户端程序都认识该选项。如果程序不理解该选项，则在尝试运行时会显示错误消息并退出。

首先列出更一般的选项组，然后再列出更具体的选项组。例如，`[client]` 组更为一般，因为所有客户端程序都会读取它，而 `[mysqldump]` 组只被 **mysqldump** 读取。后面指定的选项会覆盖先前指定的选项，因此按照 `[client]`、`[mysqldump]` 的顺序排列选项组可以使 **mysqldump** 特定的选项覆盖 `[client]` 的选项。

这是一个典型的全局选项文件：

```sql
[client]
port=3306
socket=/tmp/mysql.sock

[mysqld]
port=3306
socket=/tmp/mysql.sock
key_buffer_size=16M
max_allowed_packet=128M

[mysqldump]
quick
```

这是一个典型的用户选项文件：

```sql
[client]
# The following password is sent to all standard MySQL clients
password="my password"

[mysql]
no-auto-rehash
connect_timeout=2
```

要创建仅由特定 MySQL 版本系列的**mysqld**服务器读取的选项组，请使用名称为`[mysqld-5.7]`、`[mysqld-8.0]`等的组。以下组表示`sql_mode`设置仅适用于具有 8.0.x 版本号的 MySQL 服务器：

```sql
[mysqld-8.0]
sql_mode=TRADITIONAL
```

##### 选项文件包含

可以在选项文件中使用`!include`指令来包含其他选项文件，使用`!includedir`来搜索特定目录中的选项文件。例如，要包含`/home/mydir/myopt.cnf`文件，请使用以下指令：

```sql
!include /home/mydir/myopt.cnf
```

要搜索`/home/mydir`目录并读取其中找到的选项文件，请使用以下指令：

```sql
!includedir /home/mydir
```

MySQL 不保证按目录中选项文件的顺序读取。

注意

在 Unix 操作系统上，使用`!includedir`指令查找和包含的任何文件必须以`.cnf`结尾。在 Windows 上，此指令检查具有`.ini`或`.cnf`扩展名的文件。

编写包含的选项文件的内容与任何其他选项文件相同。也就是说，它应该包含一组选项，每个选项前面都有一个`[*`group`*]`行，指示选项适用于哪个程序。

在处理包含文件时，只使用当前程序正在查找的组中的选项。其他组将被忽略。假设`my.cnf`文件包含以下行：

```sql
!include /home/mydir/myopt.cnf
```

假设`/home/mydir/myopt.cnf`如下所示：

```sql
[mysqladmin]
force

[mysqld]
key_buffer_size=16M
```

如果`my.cnf`由**mysqld**处理，则仅使用`/home/mydir/myopt.cnf`中的`[mysqld]`组。如果文件由**mysqladmin**处理，则仅使用`[mysqladmin]`组。如果文件由任何其他程序处理，则不使用`/home/mydir/myopt.cnf`中的任何选项。

`!includedir`指令的处理方式类似，只是会读取命名目录中的所有选项文件。

如果选项文件包含`!include`或`!includedir`指令，则无论它们出现在文件中的位置如何，只要处理选项文件，就会处理这些指令命名的文件。

要使包含指令起作用，文件路径不应在引号内指定，并且不应包含转义序列。例如，在`my.ini`中提供的以下语句读取选项文件`myopts.ini`：

```sql
!include C:/ProgramData/MySQL/MySQL Server/myopts.ini
!include C:\ProgramData\MySQL\MySQL Server\myopts.ini
!include C:\\ProgramData\\MySQL\\MySQL Server\\myopts.ini
```

在 Windows 上，如果`!include *`/path/to/extra.ini`*`是文件中的最后一行，请确保在末尾添加一个换行符；否则，该行将被忽略。
