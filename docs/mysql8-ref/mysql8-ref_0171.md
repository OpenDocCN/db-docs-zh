# 6.3.4 mysqld_multi — 管理多个 MySQL 服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqld-multi.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqld-multi.html)

**mysqld_multi**旨在管理监听不同 Unix 套接字文件和 TCP/IP 端口上的连接的多个**mysqld**进程。它可以启动或停止服务器，或报告它们的当前状态。

注意

对于一些 Linux 平台，从 RPM 或 Debian 软件包安装 MySQL 包含 systemd 支持来管理 MySQL 服务器的启动和关闭。在这些平台上，因为不需要，所以不安装**mysqld_multi**。有关使用 systemd 处理多个 MySQL 实例的信息，请参阅 Section 2.5.9，“使用 systemd 管理 MySQL 服务器”。

**mysqld_multi**在`my.cnf`（或由`--defaults-file`选项指定的文件）中搜索名为`[mysqld*`N`*]`的组。*`N`*可以是任何正整数。在以下讨论中，这个数字被称为选项组号，或*`GNR`*。组号将选项组彼此区分开，并用作传递给**mysqld_multi**的参数，以指定要启动、停止或获取状态报告的服务器。在这些组中列出的选项与用于启动**mysqld**的`[mysqld]`组中使用的选项相同。（例如，参见 Section 2.9.5，“自动启动和停止 MySQL”。）但是，在使用多个服务器时，每个服务器必须使用自己的值来设置选项，例如 Unix 套接字文件和 TCP/IP 端口号。有关在多服务器环境中哪些选项必须对每个服务器唯一的更多信息，请参阅 Section 7.8，“在一台机器上运行多个 MySQL 实例”。

要调用**mysqld_multi**，请使用以下语法：

```sql
mysqld_multi [*options*] {start|stop|reload|report} [*GNR*[,*GNR*] ...]
```

`start`、`stop`、`reload`（停止和重新启动）和`report`表示要执行的操作。您可以根据接下来的*`GNR`*列表为单个服务器或多个服务器执行指定的操作。如果没有列表，**mysqld_multi**将为选项文件中的所有服务器执行操作。

每个*`GNR`*值代表一个选项组编号或一组组编号。该值应为选项文件中组名称末尾的数字。例如，名为`[mysqld17]`的组的*`GNR`*为`17`。要指定一系列数字，请用短横线分隔第一个和最后一个数字。*`GNR`*值`10-13`代表组`[mysqld10]`到`[mysqld13]`。可以在命令行上指定多个组或组范围，用逗号分隔。*`GNR`*列表中不能有空白字符（空格或制表符）；空白字符后的任何内容都将被忽略。

此命令启动单个服务器，使用选项组`[mysqld17]`：

```sql
mysqld_multi start 17
```

此命令停止多个服务器，使用选项组`[mysqld8]`和`[mysqld10]`到`[mysqld13]`：

```sql
mysqld_multi stop 8,10-13
```

要了解如何设置选项文件的示例，请使用此命令：

```sql
mysqld_multi --example
```

**mysqld_multi**按以下方式搜索选项文件：

+   使用`--no-defaults`，不会读取任何选项文件。

    | 命令行格式 | `--no-defaults` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

+   使用`--defaults-file=*`file_name`*`，只读取指定的文件。

    | 命令行格式 | `--defaults-file=filename` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

+   否则，将读取标准位置列表中的选项文件，包括由`--defaults-extra-file=*`file_name`*`选项指定的任何文件。（如果该选项多次给出，则使用最后一个值。）

    | 命令行格式 | `--defaults-extra-file=filename` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

有关这些和其他选项文件选项的更多信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

读取的选项文件将搜索`[mysqld_multi]`和`[mysqld*`N`*]`选项组。`[mysqld_multi]`组可用于为**mysqld_multi**本身设置选项。`[mysqld*`N`*]`组可用于传递给特定**mysqld**实例的选项。

`[mysqld]`或`[mysqld_safe]`组可用于所有**mysqld**或**mysqld_safe**实例共同读取的常见选项。您可以指定一个`--defaults-file=*`file_name`*`选项，以使用不同的配置文件来为该实例设置选项，在这种情况下，该文件中的`[mysqld]`或`[mysqld_safe]`组将用于该实例。

**mysqld_multi**支持以下选项。

+   `--help`

    | 命令行格式 | `--help` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示帮助消息并退出。

+   `--example`

    | 命令行格式 | `--example` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示一个示例选项文件。

+   `--log=*`file_name`*`

    | 命令行格式 | `--log=path` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `/var/log/mysqld_multi.log` |

    指定日志文件的名称。如果文件存在，则日志输出将附加到其中。

+   `--mysqladmin=*`prog_name`*`

    | 命令行格式 | `--mysqladmin=file` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    要用于停止服务器的**mysqladmin**二进制文件。

+   `--mysqld=*`prog_name`*`

    | 命令行格式 | `--mysqld=file` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    要使用的**mysqld**二进制文件。请注意，您还可以指定**mysqld_safe**作为此选项的值。如果您使用**mysqld_safe**启动服务器，您可以在相应的`[mysqld*`N`*]`选项组中包含`mysqld`或`ledir`选项。这些选项指示**mysqld_safe**应启动的服务器名称以及服务器所在目录的路径名。（请参阅第 6.3.2 节，“mysqld_safe — MySQL 服务器启动脚本”中这些选项的描述。）示例：

    ```sql
    [mysqld38]
    mysqld = mysqld-debug
    ledir  = /opt/local/mysql/libexec
    ```

+   `--no-log`

    | 命令行格式 | `--no-log` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    将日志信息打印到`stdout`而不是日志文件。默认情况下，输出会写入日志文件。

+   `--password=*`password`*`

    | 命令行格式 | `--password=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    用于调用**mysqladmin**时要使用的 MySQL 帐户的密码。请注意，与其他 MySQL 程序不同，此选项的密码值是必需的。

+   `--silent`

    | 命令行格式 | `--silent` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    静默模式；禁用警告。

+   `--tcp-ip`

    | 命令行格式 | `--tcp-ip` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    通过 TCP/IP 端口连接到每个 MySQL 服务器，而不是通过 Unix 套接字文件。 （如果套接字文件丢失，则服务器可能仍在运行，但只能通过 TCP/IP 端口访问。）默认情况下，连接是使用 Unix 套接字文件进行的。此选项会影响`stop`和`report`操作。

+   `--user=*`user_name`*`

    | 命令行格式 | `--user=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `root` |

    在调用**mysqladmin**时要使用的 MySQL 帐户的用户名。

+   `--verbose`

    | 命令行格式 | `--verbose` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    更加详细。

+   `--version`

    | 命令行格式 | `--version` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示版本信息并退出。

有关**mysqld_multi**的一些注意事项：

+   **最重要的事项**：在使用**mysqld_multi**之前，请确保您理解传递给**mysqld**服务器的选项的含义以及*为什么*您希望拥有单独的**mysqld**进程。注意使用具有相同数据目录的多个**mysqld**服务器的危险。除非你*知道*你在做什么，否则请使用单独的数据目录。在线程系统中，使用具有相同数据目录的多个服务器并不会提供额外的性能。参见 Section 7.8, “Running Multiple MySQL Instances on One Machine”。

    重要提示

    确保每个服务器的数据目录对于启动特定**mysqld**进程的 Unix 帐户是完全可访问的。*不要*使用 Unix 的 *`root`* 帐户进行此操作，除非你*知道*你在做什么。参见 Section 8.1.5, “How to Run MySQL as a Normal User”。

+   确保用于停止**mysqld**服务器（使用**mysqladmin**程序）的 MySQL 帐户对于每个服务器都具有相同的用户名和密码。此外，请确保该帐户具有`SHUTDOWN`权限。如果您要管理的服务器具有不同的管理帐户的用户名或密码，您可能希望在每个服务器上创建一个具有相同用户名和密码的帐户。例如，您可以通过为每个服务器执行以下命令来设置一个共同的`multi_admin`帐户：

    ```sql
    $> mysql -u root -S /tmp/mysql.sock -p
    Enter password:
    mysql> CREATE USER 'multi_admin'@'localhost' IDENTIFIED BY 'multipass';
    mysql> GRANT SHUTDOWN ON *.* TO 'multi_admin'@'localhost';
    ```

    参见第 8.2 节“访问控制和帐户管理”。您必须为每个**mysqld**服务器执行此操作。在连接到每个服务器时，请适当更改连接参数。请注意，帐户名的主机名部分必须允许您从要运行**mysqld_multi**的主机连接为`multi_admin`。

+   Unix 套接字文件和 TCP/IP 端口号必须对于每个**mysqld**都不同。（或者，如果主机有多个网络地址，您可以设置`bind_address`系统变量，使不同的服务器监听不同的接口。）

+   如果您使用**mysqld_safe**启动**mysqld**（例如，`--mysqld=mysqld_safe`），那么`--pid-file`选项非常重要。每个**mysqld**都应该有自己的进程 ID 文件。使用**mysqld_safe**而不是**mysqld**的优势在于，**mysqld_safe**监视其**mysqld**进程，并在进程因使用`kill -9`发送信号或由于其他原因（如分段错误）终止时重新启动它。

+   您可能想要为**mysqld**使用`--user`选项，但要这样做，您需要以 Unix 超级用户（`root`）身份运行**mysqld_multi**脚本。在选项文件中有这个选项并不重要；如果您不是超级用户并且**mysqld**进程在您自己的 Unix 帐户下启动，您只会收到警告。

以下示例显示了如何为与**mysqld_multi**一起使用的选项文件进行设置。**mysqld**程序启动或停止的顺序取决于它们在选项文件中出现的顺序。组号不需要形成连续的序列。示例中故意省略了第一个和第五个`[mysqld*`N`*]`组，以说明选项文件中可以有“间隙”。这样可以提供更多的灵活性。

```sql
# This is an example of a my.cnf file for mysqld_multi.
# Usually this file is located in home dir ~/.my.cnf or /etc/my.cnf

[mysqld_multi]
mysqld     = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /usr/local/mysql/bin/mysqladmin
user       = multi_admin
password   = my_password

[mysqld2]
socket     = /tmp/mysql.sock2
port       = 3307
pid-file   = /usr/local/mysql/data2/hostname.pid2
datadir    = /usr/local/mysql/data2
language   = /usr/local/mysql/share/mysql/english
user       = unix_user1

[mysqld3]
mysqld     = /path/to/mysqld_safe
ledir      = /path/to/mysqld-binary/
mysqladmin = /path/to/mysqladmin
socket     = /tmp/mysql.sock3
port       = 3308
pid-file   = /usr/local/mysql/data3/hostname.pid3
datadir    = /usr/local/mysql/data3
language   = /usr/local/mysql/share/mysql/swedish
user       = unix_user2

[mysqld4]
socket     = /tmp/mysql.sock4
port       = 3309
pid-file   = /usr/local/mysql/data4/hostname.pid4
datadir    = /usr/local/mysql/data4
language   = /usr/local/mysql/share/mysql/estonia
user       = unix_user3

[mysqld6]
socket     = /tmp/mysql.sock6
port       = 3311
pid-file   = /usr/local/mysql/data6/hostname.pid6
datadir    = /usr/local/mysql/data6
language   = /usr/local/mysql/share/mysql/japanese
user       = unix_user4
```

参见第 6.2.2.2 节，“使用选项文件”。
