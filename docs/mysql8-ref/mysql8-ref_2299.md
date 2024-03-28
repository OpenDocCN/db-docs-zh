> 原文：[`dev.mysql.com/doc/refman/8.0/en/can-not-connect-to-server.html`](https://dev.mysql.com/doc/refman/8.0/en/can-not-connect-to-server.html)

#### B.3.2.2 无法连接到 [local] MySQL 服务器

Unix 上的 MySQL 客户端可以通过两种不同的方式连接到 **mysqld** 服务器：通过使用 Unix 套接字文件连接到文件系统中的文件（默认 `/tmp/mysql.sock`），或者通过使用 TCP/IP，通过端口号连接。Unix 套接字文件连接比 TCP/IP 更快，但只能在连接到同一台计算机上的服务器时使用。如果不指定主机名或指定特殊主机名 `localhost`，则使用 Unix 套接字文件。

如果 MySQL 服务器在 Windows 上运行，您可以使用 TCP/IP 进行连接。如果服务器启用了 `named_pipe` 系统变量，您还可以在运行客户端的主机上使用命名管道进行连接。命名管道的名称默认为 `MySQL`。如果在连接到 **mysqld** 时未提供主机名，MySQL 客户端首先尝试连接到命名管道。如果这不起作用，它将连接到 TCP/IP 端口。您可以通过在 Windows 上使用 `.` 作为主机名来强制使用命名管道。

错误（2002）`无法连接到...` 通常意味着系统上没有运行 MySQL 服务器，或者在尝试连接到服务器时使用了不正确的 Unix 套接字文件名或 TCP/IP 端口号。您还应检查您正在使用的 TCP/IP 端口是否被防火墙或端口阻止服务阻止。

错误（2003）`无法连接到 '*`server`*' 上的 MySQL 服务器 (10061)` 表示网络连接被拒绝。您应检查是否有运行中的 MySQL 服务器，是否已启用网络连接，并且您指定的网络端口是否与服务器上配置的端口一致。

首先检查服务器主机上是否有名为 **mysqld** 的进程正在运行。（在 Unix 上使用 **ps xa | grep mysqld**，在 Windows 上使用任务管理器。）如果没有这样的进程，则应启动服务器。请参阅 Section 2.9.2, “Starting the Server”。

如果 **mysqld** 进程正在运行，您可以尝试以下命令来检查。在您的设置中，端口号或 Unix 套接字文件名可能不同。`host_ip` 代表运行服务器的机器的 IP 地址。

```sql
$> mysqladmin version
$> mysqladmin variables
$> mysqladmin -h `hostname` version variables
$> mysqladmin -h `hostname` --port=3306 version
$> mysqladmin -h host_ip version
$> mysqladmin --protocol=SOCKET --socket=/tmp/mysql.sock version
```

注意使用反引号而不是前引号与**hostname**命令一起使用；这会导致**hostname**的输出（即当前主机名）被替换为**mysqladmin**命令。如果你没有**hostname**命令或在 Windows 上运行，你可以手动输入你的机器主机名（不使用反引号）跟在`-h`选项后面。你也可以尝试使用`-h 127.0.0.1`来通过 TCP/IP 连接到本地主机。

确保服务器没有配置为忽略网络连接，或者（如果你尝试远程连接）没有配置为仅在其网络接口上本地监听。如果服务器启动时启用了`skip_networking`系统变量，它根本无法接受 TCP/IP 连接。如果服务器启动时将`bind_address`系统变量设置为`127.0.0.1`，它仅在回环接口上本地监听 TCP/IP 连接，不接受远程连接。

检查确保没有防火墙阻止访问 MySQL。你的防火墙可能根据正在执行的应用程序或 MySQL 用于通信的端口号（默认为 3306）进行配置。在 Linux 或 Unix 下，检查你的 IP 表（或类似）配置，确保端口没有被阻止。在 Windows 下，诸如 ZoneAlarm 或 Windows 防火墙等应用程序可能需要配置不要阻止 MySQL 端口。

出现`无法连接到本地 MySQL 服务器`错误的一些原因如下：

+   **mysqld**没有在本地主机上运行。检查你的操作系统进程列表，确保**mysqld**进程存在。

+   在 Windows 上运行 MySQL 服务器，并有许多 TCP/IP 连接。如果你经常遇到客户端出现这个错误，你可以在这里找到一个解决方法：第 B.3.2.2.1 节，“在 Windows 上连接到 MySQL 服务器失败”。

+   有人删除了**mysqld**使用的 Unix 套接字文件（默认为`/tmp/mysql.sock`）。例如，你可能有一个从`/tmp`目录中删除旧文件的**cron**作业。你可以随时运行**mysqladmin version**来检查**mysqladmin**正在尝试使用的 Unix 套接字文件是否真的存在。在这种情况下的修复方法是修改**cron**作业以不删除`mysql.sock`或将套接字文件放在其他地方。参见第 B.3.3.6 节，“如何保护或更改 MySQL Unix 套接字文件”。

+   你已经使用`--socket=/path/to/socket`选项启动了**mysqld**服务器，但忘记告诉客户端程序新的套接字文件名称。如果你改变了服务器的套接字路径名，你也必须通知 MySQL 客户端。你可以在运行客户端程序时提供相同的`--socket`选项来做到这一点。你还需要确保客户端有权限访问`mysql.sock`文件。要找出套接字文件的位置，你可以执行：

    ```sql
    $> netstat -ln | grep mysql
    ```

    参见第 B.3.3.6 节，“如何保护或更改 MySQL Unix 套接字文件”。

+   你正在使用 Linux 并且一个服务器线程已经死掉（核心已转储）。在这种情况下，你必须杀死其他**mysqld**线程（例如，使用**kill**）才能重新启动 MySQL 服务器。参见第 B.3.3.3 节，“如果 MySQL 一直崩溃怎么办”。

+   服务器或客户端程序可能没有适当的访问权限来访问保存 Unix 套接字文件或套接字文件本身的目录。在这种情况下，你必须要么更改目录或套接字文件的访问权限，以便服务器和客户端可以访问它们，要么使用指定在服务器可以创建它并且客户端程序可以访问它的目录中的套接字文件名称的`--socket`选项重新启动**mysqld**。

如果你收到错误消息`Can't connect to MySQL server on some_host`，你可以尝试以下方法找出问题所在：

+   通过执行 `telnet some_host 3306` 并按下回车键几次来检查服务器是否在该主机上运行。（3306 是默认的 MySQL 端口号。如果你的服务器正在监听不同的端口，请更改该值。）如果有一个 MySQL 服务器正在运行并监听该端口，你应该会收到一个包含服务器版本号的响应。如果你收到类似 `telnet: Unable to connect to remote host: Connection refused` 的错误，则表示给定端口上没有运行服务器。

+   如果服务器在本地主机上运行，请尝试使用 **mysqladmin -h localhost variables** 使用 Unix 套接字文件连接。验证服务器配置为监听的 TCP/IP 端口号（它是 `port` 变量的值）。

+   如果你在 Linux 下运行，并且启用了安全增强型 Linux（SELinux），请参见 第 8.7 节，“SELinux”。

##### B.3.2.2.1 连接到 MySQL 服务器在 Windows 上失败

当你在 Windows 上运行一个 MySQL 服务器，并且有许多 TCP/IP 连接到它，而且经常出现你的客户端经常出现 `Can't connect to MySQL server` 错误时，原因可能是 Windows 不允许足够的短暂（短暂的）端口来服务这些连接。

`TIME_WAIT` 的目的是在连接关闭后保持接受数据包的连接。这是因为互联网路由可能导致数据包以缓慢的速度到达目的地，可能在双方都同意关闭后到达。如果端口用于新连接，来自旧连接的数据包可能会破坏协议或泄露原始连接的个人信息。`TIME_WAIT` 延迟通过确保端口在允许延迟数据包到达一段时间后才能被重用，从而防止这种情况发生。

在局域网连接上大大减少 `TIME_WAIT` 是安全的，因为在这种情况下，包到达的延迟很小，而通过互联网，由于相对较大的距离和延迟，包可能会有很长的延迟。

Windows 允许用户使用短暂（短暂的）TCP 端口。在任何端口关闭后，它将保持在 `TIME_WAIT` 状态 120 秒。在此时间到期之前，该端口将不再可用。端口号的默认范围取决于 Windows 的版本，旧版本中的端口数量较少：

+   Windows Server 2003 及之前版本：端口范围在 1025–5000

+   Windows Vista、Server 2008 和更新版本：端口范围在 49152–65535

有一个可用 TCP 端口堆栈（5000）和在短时间内打开和关闭大量 TCP 端口以及 `TIME_WAIT` 状态，你很有可能耗尽端口。解决这个问题有两种方法：

+   通过调查连接池或尽可能使用持久连接来快速减少消耗的 TCP 端口数量

+   调整 Windows 注册表中的一些设置（见下文）

重要

以下过程涉及修改 Windows 注册表。在修改注册表之前，请确保备份并确保了解如何在出现问题时恢复它。有关如何备份、恢复和编辑注册表的信息，请查看 Microsoft 知识库中的以下文章：[`support.microsoft.com/kb/256986/EN-US/`](http://support.microsoft.com/kb/256986/EN-US/)

1.  启动注册表编辑器（`Regedt32.exe`）。

1.  在注册表中找到以下键：

    ```sql
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
    ```

1.  在`编辑`菜单上，单击`添加值`，然后添加以下注册表值：

    ```sql
    Value Name: MaxUserPort
    Data Type: REG_DWORD
    Value: 65534
    ```

    这将设置任何用户可用的临时端口数量。有效范围为 5000 到 65534（十进制）。默认值为 0x1388（5000 十进制）。

1.  在`编辑`菜单上，单击`添加值`，然后添加以下注册表值：

    ```sql
    Value Name: TcpTimedWaitDelay
    Data Type: REG_DWORD
    Value: 30
    ```

    这将设置在关闭之前保持 TCP 端口连接处于`TIME_WAIT`状态的秒数。有效范围为 30 到 300 十进制，尽管您可能希望与微软核实最新允许的值。默认值为 0x78（120 十进制）。

1.  退出注册表编辑器。

1.  重新启动计算机。

注意：撤消上述操作应该就像删除您创建的注册表条目一样简单。
