> 原文：[`dev.mysql.com/doc/refman/8.0/en/MySQLInstallerConsole.html`](https://dev.mysql.com/doc/refman/8.0/en/MySQLInstallerConsole.html)

#### 2.3.3.5 MySQL Installer ���制台参考

**MySQLInstallerConsole.exe**提供类似于 MySQL Installer 的命令行功能。此参考包括：

+   MySQL 产品名称

+   命令语法

+   命令操作

当首次执行 MySQL Installer 时，控制台会被安装，然后在`MySQL Installer for Windows`目录中可用。默认情况下，目录位置为`C:\Program Files (x86)\MySQL\MySQL Installer for Windows`。您必须以管理员身份运行控制台。

要使用控制台：

1.  通过从“开始”中选择“Windows 系统”，右键单击“命令提示符”，选择“更多”，然后选择“以管理员身份运行”来打开具有管理员权限的命令提示符。

1.  从命令行，可选择将目录更改为**MySQLInstallerConsole.exe**命令所在的位置。例如，要使用默认安装位置：

    ```sql
    cd Program Files (x86)\MySQL\MySQL Installer for Windows
    ```

1.  输入`MySQLInstallerConsole.exe`（或`mysqlinstallerconsole`），然后跟随一个命令操作来执行任务。例如，要显示控制台的帮助：

    ```sql
    MySQLInstallerConsole.exe --help
    ```

    ```sql
    =================== Start Initialization ===================
    MySQL Installer is running in Community mode

    Attempting to update manifest.
    Initializing product requirements.
    Loading product catalog.
    Checking for product packages in the bundle.
    Categorizing product catalog.
    Finding all installed packages.
    Your product catalog was last updated at 23/08/2022 12:41:05 p. m.
    Your product catalog has version number 671.
    =================== End Initialization ===================

    The following actions are available:

    Configure - Configures one or more of your installed programs.
    Help      - Provides list of available command actions.
    Install   - Installs and configures one or more available MySQL programs.
    List      - Lists all available MySQL products.
    Modify    - Modifies the features of installed products.
    Remove    - Removes one or more products from your system.
    Set       - Configures the general options of MySQL Installer.
    Status    - Shows the status of all installed products.
    Update    - Updates the current product catalog.
    Upgrade   - Upgrades one or more of your installed programs.

    The basic syntax for using MySQL Installer command actions. Brackets denote optional entities. 
    Curly braces denote a list of possible entities.

    ...

    ```

##### MySQL 产品名称

许多**MySQLInstallerConsole**命令操作接受一个或多个缩写短语，这些短语可以匹配目录中的 MySQL 产品（或产品）。用于命令的当前一组有效的短语在下表中显示。

注意

从 MySQL Installer 1.6.7 (8.0.34) 开始，`install`、`list` 和 `upgrade` 命令选项不再适用于 MySQL for Visual Studio（现已停用）、MySQL 连接器/NET、MySQL 连接器/ODBC、MySQL 连接器/C++、MySQL 连接器/Python 和 MySQL 连接器/J。要安装更新的 MySQL 连接器，请访问 https://dev.mysql.com/downloads/。

**表 2.6 用于 MySQLInstallerConsole.exe 命令的 MySQL 产品短语**

| 短语 | MySQL 产品 |
| --- | --- |
| `server` | MySQL 服务器 |
| `workbench` | MySQL Workbench |
| `shell` | MySQL Shell |
| `visual` | MySQL for Visual Studio |
| `router` | MySQL 路由器 |
| `backup` | MySQL 企业备份（需要商业版本） |
| `net` | MySQL 连接器/NET |
| `odbc` | MySQL 连接器/ODBC |
| `c++` | MySQL 连接器/C++ |
| `python` | MySQL 连接器/Python |
| `j` | MySQL 连接器/J |
| `documentation` | MySQL 服务器文档 |
| `samples` | MySQL 示例（sakila 和 world 数据库） |
| 短语 | MySQL 产品 |

##### 命令语法

**MySQLInstallerConsole.exe** 命令可以带有或不带有文件扩展名（`.exe`）发出，并且命令不区分大小写。

`mysqlinstallerconsole`[`.exe`] [[[`--`]*`action`*] [*`action_blocks_list`*] [*`options_list`*]]

描述：

`*`action`*`

允许的操作行为之一。如果省略，则默认操作等同于 `--status` 操作。对于所有操作，使用 `--` 前缀是可选的。

可能的操作有：[--]`configure`、[--]`help`、[--]`install`、[--]`list`、[--]`modify`、[--]`remove`、[--]`set`、[--]`status`、[--]`update` 和 [--]`upgrade`。

`*`action_blocks_list`*`

一个块列表，每个块代表一个不同的项目，具体取决于所选操作。块之间用逗号分隔。

`--remove` 和 `--upgrade` 操作允许指定星号字符（`*`）表示所有产品。如果在此块的开头检测到 `*` 字符，则假定要处理所有产品，并忽略该块的其余部分。

语法：`*|*`action_block`*[,*`action_block`*][,*`action_block`*]...`

*`action_block`*：包含产品选择器，后跟数量不定的参数块，这些参数块根据所选操作的不同而表现不同（参见 Command Actions）。

`*`options_list`*`

零个或多个带有可能值的选项，用空格分隔。请参阅 Command Actions 以确定对应操作允许的选项。

语法：`*`option_value_pair`*[ *`option_value_pair`*][ *`option_value_pair`*]...`

*`option_value_pair`*：单个选项（例如，`--silent`）或具有选项前缀的键和相应值的元组。键值对的形式为 `--*`key`*[=*`value`*]`。

##### 命令操作

**MySQLInstallerConsole.exe** 支持以下命令操作：

注意

包含冒号字符（`:`）的配置块（或参数块）值必须用引号括起来。例如，`install_dir="C:\MySQL\MySQL Server 8.0"`。

+   `[--]configure [*`product1`*]:[*`configuration_argument`*]=[*`value`*], [*`product2`*]:[*`configuration_argument`*]=[*`value`*], [*`...`*]`

    配置系统上的一个或多个 MySQL 产品。每个产品可以配置多个 *`configuration_argument`*=*`value`* 对。

    选项：

    `--continue`

    在处理包含每个产品参数的操作块时捕获错误时，继续处理下一个产品。如果未指定，在出现错误时整个操作将被中止。

    `--help`

    显示相应操作的选项和可用参数。如果存在该操作不会被执行，只会显示帮助信息，因此其他与操作相关的选项也会被忽略。

    `--show-settings`

    通过在`--show-settings`后传入产品名称来显示所选产品的可用选项。

    `--silent`

    禁用确认提示。

    示例：

    ```sql
    MySQLInstallerConsole --configure --show-settings server
    ```

    ```sql
    mysqlinstallerconsole.exe --configure server:port=3307
    ```

+   `[--]help`

    显示带有用法示例的帮助消息，然后退出。传入额外的命令操作以接收特定于该操作的帮助。

    选项：

    `--action=*`[action]`*`

    显示特定操作的帮助信息。与使用带有操作的`--help`选项相同。

    允许的值为：`all`、`configure`、`help`（默认）、`install`、`list`、`modify`、`remove`、`status`、`update`、`upgrade`和`set`。

    `--help`

    显示相应操作的选项和可用参数。如果存在该操作，则仅显示帮助信息，不执行操作，因此其他与操作相关的选项也将被忽略。

    示例：

    ```sql
    MySQLInstallerConsole help
    ```

    ```sql
    MySQLInstallerConsole help --action=install
    ```

+   `[--]install [*`产品 1`*]:[*`功能`*]:[*`配置块`*]:[*`配置块`*]，[*`产品 2`*]:[*`配置块`*]，[*`...`*]`

    在系统上安装一个或多个 MySQL 产品。如果有预发布产品可用，当`--type`选项值为`Client`或`Full`时，将安装 GA 和预发布产品。在使用这些设置类型时，使用`--only_ga_products`选项将产品集限制为仅 GA 产品。

    描述：

    `[*`产品`*]`

    每个产品可以通过产品短语来指定，可以带有分号分隔的版本限定符，也可以不带。仅传入产品关键字会选择产品的最新版本。如果该版本的产品有多个架构可用，命令将返回清单列表中的第一个架构以供交互式确认。或者，您可以在产品关键字后使用`--silent`选项传入确切的版本和架构（`x86`或`x64`）。

    `[*`功能`*]`

    默认情况下安装与 MySQL 产品关联的所有功能。功能块是一个以分号分隔的功能列表或选择所有功能的星号字符（`*`）。要移除功能，请使用`modify`命令。

    `[*`配置块`*]`

    可指定一个或多个配置块。每个配置块是一个以分号分隔的键值对列表。一个块可以包含`config`或`user`类型键；如果未定义，则`config`是默认类型。

    包含冒号字符（`:`）的配置块值必须用引号括起来。例如，`installdir="C:\MySQL\MySQL Server 8.0"`。每个产品只能定义一个配置类型块。在产品安装期间应为要创建的每个用户定义一个用户块。

    注意

    当重新配置产品时，不支持`user`类型键。

    选项：

    `--auto-handle-prereqs`

    如果存在，MySQL 安装程序会尝试下载和安装一些当前不存在的软件先决条件，可以通过最小干预解决。如果未使用`--silent`选项，则会为每个先决条件显示安装页面。如果省略`--auto-handle-prereqs`选项，则不会安装缺少先决条件的软件包。

    `--continue`

    在处理每个产品的参数块时捕获错误时，继续处理下一个产品。如果未指定，则在出现错误时整个操作将中止。

    `--help`

    显示相应操作的选项和可用参数。如果存在该选项，则仅显示帮助，不执行操作，因此其他与操作相关的选项也将被忽略。

    `--mos-password=*`password`*`

    设置 My Oracle Support（MOS）用户的商业版 MySQL 安装程序密码。

    `--mos-user=*`user_name`*`

    指定用于访问商业版 MySQL 安装程序的 My Oracle Support（MOS）用户名。如果不存在，则仅可安装捆绑包中的产品（如果有）。

    `--only-ga-products`

    限制产品集以仅包括 GA 产品。

    `--setup-type=*`setup_type`*`

    安装预定义的软件集。设置类型可以是以下之一：

    +   `Server`: 安装单个 MySQL 服务器

    +   `Client`: 安装客户端程序和库（不包括 MySQL 连接器）

    +   `Full`: 安装所有内容（不包括 MySQL 连接器）

    +   `Custom`: 安装用户选择的产品。这是默认选项。

    注意

    仅当未安装其他 MySQL 产品时，非自定义设置类型才有效。

    `--show-settings`

    通过在`-showsettings`后传入产品名称，显示所选产品的可用选项。

    `--silent`

    禁用确认提示。

    示例：

    ```sql
    mysqlinstallerconsole.exe --install j;8.0.29, net;8.0.28 --silent
    ```

    ```sql
    MySQLInstallerConsole install server;8.0.30:*:port=3307;server_id=2:type=user;user=foo
    ```

    一个示例，通过`^`分隔传入额外的配置块以适应：

    ```sql
    MySQLInstallerConsole --install server;8.0.30;x64:*:type=config;open_win_firewall=true; ^
       general_log=true;bin_log=true;server_id=3306;tcp_ip=true;port=3306;root_passwd=pass; ^
       install_dir="C:\MySQL\MySQL Server 8.0":type=user;user_name=foo;password=bar;role=DBManager
    ```

+   `[--]list`

    当此操作不带选项使用时，会激活一个交互式列表，可搜索所有可用的 MySQL 产品。输入`MySQLInstallerConsole --list`并指定要搜索的子字符串。

    选项：

    `--all`

    列出所有可用产品。如果使用此选项，则忽略所有其他选项。

    `--arch=*`architecture`*`

    列出包含指定架构的产品。允许的值为：`x86`，`x64` 和 `any`（默认）。此选项可与`--name`和`--version`选项结合使用。

    `--help`

    显示相应操作的选项和可用参数。如果存在该选项，则仅显示帮助，不执行操作，因此其他与操作相关的选项也将被忽略。

    `--name=*`package_name`*`

    列出包含指定名称的产品（参见产品短语），此选项可与`--version`和`--arch`选项结合使用。

    `--version=*`version`*`

    列出包含指定版本（如 8.0 或 5.7）的产品。此选项可以与 `--name` 和 `--arch` 选项结合使用。

    示例：

    ```sql
    MySQLInstallerConsole --list --name=net --version=8.0
    ```

+   `[--]modify [*`product1`*:-*`removelist`*|+*`addlist`*], [*`product2`*:-*`removelist`*|+*`addlist`*] [*`...`*]

    修改或显示先前安装的 MySQL 产品的特性。要显示产品的特性，请将产品关键字附加到命令后，例如：

    ```sql
    MySQLInstallerConsole --modify server
    ```

    选项：

    `--help`

    显示相应操作的选项和可用参数。如果存在，操作不会被执行，只会显示帮助信息，因此其他与操作相关的选项也会被忽略。

    `--silent`

    禁用确认提示。

    示例：

    ```sql
    MySQLInstallerConsole --modify server:+documentation
    ```

    ```sql
    MySQLInstallerConsole modify server:-debug
    ```

+   `[--]remove [*`product1`*], [*`product2`*] [*`...`*]`

    从系统中移除一个或多个产品。可以传入星号字符（`*`）以一条命令移除所有 MySQL 产品。

    选项：

    `--continue`

    即使发生错误也继续操作。

    `--help`

    显示相应操作的选项和可用参数。如果存在，操作不会被执行，只会显示帮助信息，因此其他与操作相关的选项也会被忽略。

    `--keep-datadir`

    在删除 MySQL Server 产品时跳过数据目录的移除。

    `--silent`

    禁用确认提示。

    示例：

    ```sql
    mysqlinstallerconsole.exe remove *
    ```

    ```sql
    MySQLInstallerConsole --remove server --continue
    ```

+   `[--]set`

    设置一个或多个可配置选项，影响 MySQL 安装程序连接到互联网的方式以及是否激活自动产品目录更新功能。

    选项：

    `--catalog-update=*`bool_value`*`

    启用（`true`，默认）或禁用（`false`）自动产品目录更新。此选项需要与互联网的活动连接。

    `--catalog-update-days=*`int_value`*`

    接受介于 1（默认）和 365 之间的整数，表示 MySQL 安装程序启动时检查新目录更新之间的天数。如果 `--catalog-update` 为 `false`，则此选项将被忽略。

    `--connection-validation=*`validation_type`*`

    设置 MySQL 安装程序如何执行对互联网连接的检查。允许的值为 `automatic`（默认）和 `manual`。

    `--connection-validation-urls=*`url_list`*`

    用双引号括起来并以逗号分隔的字符串，定义了在 `--connection-validation` 设置为 `manual` 时用于检查互联网连接的 URL 列表。按照提供的顺序进行检查。如果第一个 URL 失败，则使用列表中的下一个 URL，依此类推。

    `--offline-mode=*`bool_value`*`

    启用 MySQL 安装程序以具有或不具有互联网功能运行。有效模式包括：

    +   `True` 以启用离线模式（无需互联网连接运行）。

    +   `False`（默认）以禁用离线模式（需要互联网连接运行）。在下载产品目录或任何要安装的产品之前设置此模式。

    `--proxy-mode`

    指定代理模式。有效模式包括：

    +   `Automatic` 以根据系统设置自动识别代理。

    +   `None` 以确保未配置代理。

    +   `Manual` 手动设置代理详细信息（`--proxy-server`、`--proxy-port`、`--proxy-username`、`--proxy-password`）。

    `--proxy-password`

    用于认证到代理服务器的密码。

    `--proxy-port`

    代理服务器使用的端口。

    `--proxy-server`

    指向代理服务器的 URL。

    `--proxy-username`

    用于认证到代理服务器的用户名。

    `--reset-defaults`

    将与`--set`操作相关联的 MySQL Installer 选项重置为默认值。

    示例：

    ```sql
    MySQLIntallerConsole.exe set --reset-defaults
    ```

    ```sql
    mysqlintallerconsole.exe --set --catalog-update=false
    ```

    ```sql
    MySQLIntallerConsole --set --catalog-update-days=3
    ```

    ```sql
    mysqlintallerconsole --set --connection-validation=manual 
    --connection-validation-urls="https://www.bing.com,http://www.google.com"
    ```

+   `[--]status`

    提供系统上已安装的 MySQL 产品的快速概述。信息包括产品名称和版本、架构、安装日期和安装位置。

    选项：

    `--help`

    显示相应操作的选项和可用参数。如果存在，仅显示帮助，不执行操作，因此其他与操作相关的选项也将被忽略。

    示例：

    ```sql
    MySQLInstallerConsole status
    ```

+   `[--]update`

    将最新的 MySQL 产品目录下载到您的系统。成功后，目录将在下次执行`MySQLInstaller`或**MySQLInstallerConsole.exe**时应用。

    当距离上次检查已经过去*`n`*天时，MySQL Installer 在启动时会自动检查产品目录更新。从 MySQL Installer 1.6.4 开始，默认值为 1 天。之前的默认值为 7 天。

    选项：

    `--help`

    显示相应操作的选项和可用参数。如果存在，仅显示帮助，不执行操作，因此其他与操作相关的选项也将被忽略。

    示例：

    ```sql
    MySQLInstallerConsole update
    ```

+   `[--]upgrade [*`product1`*:*`version`*], [*`product2`*:*`version`*] [*`...`*]`

    升级系统上的一个或多个产品。此操作允许使用以下字符：

    `*`

    传入`*`以将所有产品升级到最新版本，或传入特定产品。

    `!`

    将`!`作为版本号传入，将 MySQL 产品升级到最新版本。

    选项：

    `--continue`

    继续操作，即使发生错误。

    `--help`

    显示相应操作的选项和可用参数。如果存在，仅显示帮助，不执行操作，因此其他与操作相关的选项也将被忽略。

    `--mos-password=*`password`*`

    设置商业版本 MySQL Installer 的 My Oracle Support (MOS) 用户密码。

    `--mos-user=*`user_name`*`

    指定用于访问 MySQL Installer 商业版本的 My Oracle Support (MOS) 用户名称。如果不存在，只有捆绑包中的产品可供安装。

    `--silent`

    禁用确认提示。

    示例：

    ```sql
    MySQLInstallerConsole upgrade *
    ```

    ```sql
    MySQLInstallerConsole upgrade workbench:8.0.31
    ```

    ```sql
    MySQLInstallerConsole upgrade workbench:!
    ```

    ```sql
    MySQLInstallerConsole --upgrade server;8.0.30:!, j;8.0.29:!
    ```
