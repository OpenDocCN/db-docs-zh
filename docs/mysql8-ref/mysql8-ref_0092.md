# 2.9.1 初始化数据目录

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html`](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html)

安装 MySQL 后，必须初始化数据目录，包括`mysql`系统模式中的表：

+   对于某些 MySQL 安装方法，数据目录初始化是自动的，如第 2.9 节，“安装后设置和测试”中所述。

+   对于其他安装方法，您必须手动初始化数据目录。这些方法包括在 Unix 和类 Unix 系统上从通用二进制和源分发安装，以及在 Windows 上从 ZIP 存档包安装。

本节描述了如何手动为 MySQL 安装方法初始化数据目录，其中数据目录初始化不是自动的。有一些建议的命令可用于测试服务器是否可访问和正常工作，请参见第 2.9.3 节，“测试服务器”。

注意

在 MySQL 8.0 中，默认身份验证插件已从`mysql_native_password`更改为`caching_sha2_password`，并且`'root'@'localhost'`管理帐户默认使用`caching_sha2_password`。如果您希望`root`帐户使用先前的默认身份验证插件（`mysql_native_password`），请参见 caching_sha2_password 和 root 管理帐户。

+   数据目录初始化概述

+   数据目录初始化过程

+   数据目录初始化期间的服务器操作

+   初始化后的 root 密码分配

#### 数据目录初始化概述

在这里显示的示例中，服务器旨在在`mysql`登录帐户的用户 ID 下运行。如果该帐户不存在，请创建该帐户（参见创建 mysql 用户和组），或者替换您计划用于运行服务器的不同现有登录帐户的名称。

1.  将位置更改为 MySQL 安装的顶级目录，通常为`/usr/local/mysql`（根据需要调整系统的路径名）：

    ```sql
    cd /usr/local/mysql
    ```

    在此目录中，您可以找到几个文件和子目录，包括包含服务器以及客户端和实用程序程序的`bin`子目录。

1.  `secure_file_priv`系统变量限制导入和导出操作到特定目录。创建一个目录，其位置可以指定为该变量的值：

    ```sql
    mkdir mysql-files
    ```

    将目录的用户和组所有权授予`mysql`用户和`mysql`组，并适当设置目录权限：

    ```sql
    chown mysql:mysql mysql-files
    chmod 750 mysql-files
    ```

1.  使用服务器初始化数据目录，包括包含初始 MySQL 授权表的`mysql`模式，这些表确定用户如何连接到服务器。例如：

    ```sql
    bin/mysqld --initialize --user=mysql
    ```

    对于命令的重要信息，特别是关于您可能使用的命令选项，请参阅数据目录初始化过程。有关服务器执行初始化的详细信息，请参阅数据目录初始化期间的服务器操作。

    通常，只有在首次安装 MySQL 后才需要进行数据目录初始化。（对于现有安装的升级，请执行升级过程；请参阅第三章，*升级 MySQL*。）但是，初始化数据目录的命令不会覆盖任何现有的`mysql`模式表，因此在任何情况下运行都是安全的。

1.  如果您希望部署具有自动支持安全连接的服务器，请使用**mysql_ssl_rsa_setup**实用程序创建默认的 SSL 和 RSA 文件：

    ```sql
    bin/mysql_ssl_rsa_setup
    ```

    欲了解更多信息，请参阅 6.4.3 节，“mysql_ssl_rsa_setup — 创建 SSL/RSA 文件”。

    注意

    **mysql_ssl_rsa_setup**实用程序已在 MySQL 8.0.34 中弃用。

1.  在没有任何选项文件的情况下，服务器将使用默认设置启动。（请参阅 7.1.2 节，“服务器配置默认值”。）要明确指定 MySQL 服务器在启动时应使用的选项，请将它们放在选项文件中，例如`/etc/my.cnf`或`/etc/mysql/my.cnf`。（请参阅 6.2.2.2 节，“使用选项文件”。）例如，您可以使用选项文件设置`secure_file_priv`系统变量。

1.  要安排 MySQL 在系统启动时无需手动干预即可启动，请参阅第 2.9.5 节“自动启动和停止 MySQL”。

1.  数据目录初始化在`mysql`模式中创建时区表，但不填充它们。要执行此操作，请使用第 7.1.15 节“MySQL 服务器时区支持”中的说明。

#### 数据目录初始化过程

切换到 MySQL 安装的顶级目录，通常为`/usr/local/mysql`（根据需要调整系统的路径名）：

```sql
cd /usr/local/mysql
```

要初始化数据目录，请使用`--initialize`或`--initialize-insecure`选项调用**mysqld**，具体取决于您是否希望服务器为`'root'@'localhost'`帐户生成一个随机初始密码，或者创建该帐户而不设置密码：

+   使用`--initialize`进行“默认安全”安装（包括生成随机初始`root`密码）。在这种情况下，密码被标记为过期，您必须选择一个新密码。

+   使用`--initialize-insecure`，不会生成`root`密码。这是不安全的；假定您打算在将服务器��入生产使用之前及时为该帐户分配密码。

要了解如何分配新的`'root'@'localhost'`密码，请参阅数据目录初始化后的 root 密码分配。

注意

服务器将任何消息（包括任何初始密码）写入其标准错误输出。如果您在屏幕上看不到消息，请查看错误日志。有关错误日志的信息，包括其位置，请参阅第 7.4.2 节“错误日志”。

在 Windows 上，使用`--console`选项将消息重定向到控制台。

在 Unix 和类 Unix 系统上，数据库目录和文件的所有权归`mysql`登录帐户所有，以便在以后运行时服务器可以读取和写入这些文件。为了确保这一点，从系统`root`帐户启动**mysqld**，并包括如下所示的`--user`选项：

```sql
bin/mysqld --initialize --user=mysql
bin/mysqld --initialize-insecure --user=mysql
```

或者，以`mysql`身份登录时执行**mysqld**，在这种情况下，您可以从命令中省略`--user`选项。

在 Windows 上，使用以下命令之一：

```sql
bin\mysqld --initialize --console
bin\mysqld --initialize-insecure --console
```

注意

如果缺少必需的系统库，数据目录初始化可能会失败。例如，您可能会看到类似于以下错误：

```sql
bin/mysqld: error while loading shared libraries:
libnuma.so.1: cannot open shared object file:
No such file or directory
```

如果发生这种情况，您必须手动安装缺失的库或使用系统的软件包管理器。然后重试数据目录初始化命令。

如果**mysqld**无法识别安装目录或数据目录的正确位置，则可能需要指定其他选项，例如`--basedir`或`--datadir`。例如（将命令输入在一行上）：

```sql
bin/mysqld --initialize --user=mysql
  --basedir=/opt/mysql/mysql
  --datadir=/opt/mysql/mysql/data
```

或者，将相关的选项设置放入一个选项文件中，并将该文件的名称传递给**mysqld**。对于 Unix 和类 Unix 系统，假设选项文件名为`/opt/mysql/mysql/etc/my.cnf`。将以下内容放入文件中：

```sql
[mysqld]
basedir=/opt/mysql/mysql
datadir=/opt/mysql/mysql/data
```

然后按照以下方式调用**mysqld**（将命令输入在一行上，首先使用`--defaults-file`选项）：

```sql
bin/mysqld --defaults-file=/opt/mysql/mysql/etc/my.cnf
  --initialize --user=mysql
```

在 Windows 上，假设`C:\my.ini`包含以下内容：

```sql
[mysqld]
basedir=C:\\Program Files\\MySQL\\MySQL Server 8.0
datadir=D:\\MySQLdata
```

然后按照以下方式调用**mysqld**（同样，您应该将命令输入在一行上，首先使用`--defaults-file`选项）：

```sql
bin\mysqld --defaults-file=C:\my.ini
   --initialize --console
```

重要

在初始化数据目录时，您不应指定除用于设置目录位置的选项（如`--basedir`或`--datadir`）以及如果需要的话`--user`选项之外的任何选项。在初始化后重新启动 MySQL 服务器时可以设置 MySQL 服务器在正常使用期间要使用的选项。有关更多信息，请参阅`--initialize`选项的描述。

#### 数据目录初始化期间服务器执行的操作

注意

服务器执行的数据目录初始化序列不会替代**mysql_secure_installation**和**mysql_ssl_rsa_setup**执行的操作。请参阅 Section 6.4.2, “mysql_secure_installation — Improve MySQL Installation Security”，以及 Section 6.4.3, “mysql_ssl_rsa_setup — Create SSL/RSA Files”。

当使用`--initialize`或`--initialize-insecure`选项调用**mysqld**时，在数据目录初始化序列期间执行以下操作：

1.  服务器检查数据目录的存在如下：

    +   如果不存在数据目录，则服务器将创建它。

    +   如果数据目录存在但不为空（即包含文件或子目录），服务器在生成错误消息后退出：

        ```sql
        [ERROR] --initialize specified but the data directory exists. Aborting.
        ```

        在这种情况下，删除或重命名数据目录，然后重试。

        如果现有数据目录允许非空，但每个条目的名称都以句点（`.`）开头。

1.  在数据目录中，服务器创建`mysql`系统模式及其表，包括数据字典表、授权表、时区表和服务器端帮助表。请参阅 Section 7.3, “The mysql System Schema”。

1.  服务器初始化系统表空间和管理`InnoDB`表所需的相关数据结构。

    注意

    在**mysqld**设置`InnoDB`系统表空间后，对表空间特性的某些更改需要设置一个全新的实例。符合更改的包括系统表空间中第一个文件的文件名和撤销日志的数量。如果不想使用默认值，请确保在运行**mysqld***之前*在 MySQL 配置文件中设置`innodb_data_file_path`和`innodb_log_file_size`配置参数。还要确保根据需要指定其他影响`InnoDB`文件创建和位置的参数，如`innodb_data_home_dir`和`innodb_log_group_home_dir`。

    如果这些选项在您的配置文件中，但该文件不在 MySQL 默认读取的位置，请在运行**mysqld**时使用`--defaults-extra-file`选项指定文件位置。

1.  服务器创建一个`'root'@'localhost'`超级用户账户和其他保留账户（参见第 8.2.9 节，“保留账户”）。一些保留账户被锁定，不能被客户端使用，但`'root'@'localhost'`用于管理目的，您应该为其分配一个密码。

    关于`'root'@'localhost'`账户的密码，服务器的操作取决于您如何调用它：

    +   使用`--initialize`但不使用`--initialize-insecure`，服务器生成一个随机密码，标记为过期，并显示密码的消息：

        ```sql
        [Warning] A temporary password is generated for root@localhost:
        iTag*AfrH5ej
        ```

    +   使用`--initialize-insecure`（无论是否使用`--initialize`，因为`--initialize-insecure`隐含了`--initialize`），服务器不生成密码或标记为过期，并写入警告消息：

        ```sql
        [Warning] root@localhost is created with an empty password ! Please
        consider switching off the --initialize-insecure option.
        ```

    有关分配新的`'root'@'localhost'`密码的说明，请参见初始化后的 root 密码分配。

1.  服务器填充用于`HELP`语句的服务器端帮助表（参见第 15.8.3 节，“HELP 语句”）。服务器不填充时区表。要手动执行此操作，请参见第 7.1.15 节，“MySQL 服务器时区支持”。

1.  如果`init_file`系统变量指定了一个包含 SQL 语句的文件，服务器将执行文件中的语句。此选项使您能够执行自定义引导序列。

    当服务器以引导模式运行时，某些功能不可用，限制了文件中允许的语句。这些包括与账户管理相关的语句（如`CREATE USER`或`GRANT`)、复制和全局事务标识符。

1.  服务器退出。

#### 初始化后的 root 密码分配

在使用`--initialize`或`--initialize-insecure`启动服务器初始化数据目录后，正常启动服务器（即，不使用这两个选项之一）并为`'root'@'localhost'`账户分配一个新密码：

1.  启动服务器。有关说明，请参见第 2.9.2 节，“启动服务器”。

1.  连接到服务器：

    +   如果您使用了`--initialize`但没有使用`--initialize-insecure`来初始化数据目录，请以`root`身份连接到服务器：

        ```sql
        mysql -u root -p
        ```

        然后，在密码提示符处，输入服务器在初始化序列期间生成的随机密码：

        ```sql
        Enter password: *(enter the random root password here)*
        ```

        如果您不知道此密码，请查看服务器错误日志。

    +   如果您使用了`--initialize-insecure`来初始化数据目录，请以无密码的`root`身份连接到服务器：

        ```sql
        mysql -u root --skip-password
        ```

1.  连接后，使用`ALTER USER`语句分配一个新的`root`密码：

    ```sql
    ALTER USER 'root'@'localhost' IDENTIFIED BY '*root-password'*;
    ```

请参阅 第 2.9.4 节，“保护初始 MySQL 帐户”。

注意

尝试连接到主机`127.0.0.1`通常会解析为`localhost`帐户。但是，如果服务器启用了`skip_name_resolve`，则会失败。如果您打算这样做，请确保存在一个可以接受连接的帐户。例如，要能够使用`--host=127.0.0.1`或`--host=::1`连接为`root`，请创建这些帐户：

```sql
CREATE USER 'root'@'127.0.0.1' IDENTIFIED BY '*root-password*';
CREATE USER 'root'@'::1' IDENTIFIED BY '*root-password*';
```

可以将这些语句放在一个文件中，通过`init_file`系统变量执行，如数据目录初始化期间的服务器操作中所讨论的那样。
