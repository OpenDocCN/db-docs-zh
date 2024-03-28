# 2.2 在 Unix/Linux 上使用通用二进制文件安装 MySQL

> 译文：[`dev.mysql.com/doc/refman/8.0/en/binary-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html)

Oracle 提供了一组 MySQL 的二进制发行版。这些包括各种平台的通用二进制发行版，以压缩的**tar**文件形式（扩展名为`.tar.xz`的文件），以及针对选定平台的特定平台包格式的二进制文件。

本节涵盖了在 Unix/Linux 平台上从压缩**tar**文件二进制发行版安装 MySQL 的过程。有关以 MySQL 安全功能为重点的 Linux 通用二进制发行版安装说明，请参阅安全部署指南。有关其他特定平台的二进制包格式，请参阅本手册中的其他特定平台部分。例如，对于 Windows 发行版，请参阅第 2.3 节，“在 Microsoft Windows 上安装 MySQL”。请参阅第 2.1.3 节，“如何获取 MySQL”以获取不同发行格式的 MySQL。

MySQL 压缩**tar**文件二进制发行版的名称形式为`mysql-*`VERSION`*-*`OS`*.tar.xz`，其中`*`VERSION`*`是一个数字（例如，`8.0.36`），而*`OS`*表示发行版所针对的操作系统类型（例如，`pc-linux-i686`或`winx64`）。

Linux 通用二进制发行版还有一个“最小安装”版本的 MySQL 压缩**tar**文件，其名称形式为`mysql-*`VERSION`*-*`OS`*-*`GLIBCVER`*-*`ARCH`*-minimal.tar.xz。最小安装发行版不包括调试二进制文件，并且剥离了调试符号，使其比常规二进制发行版要小得多。如果您选择安装最小安装发行版，请记得在接下来的说明中调整文件名格式的差异。

警告

+   如果您以前使用操作系统本机包管理系统（如 Yum 或 APT）安装过 MySQL，则可能在使用本机二进制文件安装时遇到问题。确保您之前安装的 MySQL 已完全删除（使用您的包管理系统），并且任何其他文件，如旧版本的数据文件，也已删除。您还应检查配置文件，如`/etc/my.cnf`或`/etc/mysql`目录，并将其删除。

    有关使用官方 MySQL 包替换第三方包的信息，请参阅相关的 APT 指南或 Yum 指南。

+   MySQL 依赖于 `libaio` 库。如果本地未安装此库，则数据目录初始化和后续服务器启动步骤将失败。如果需要，使用适当的软件包管理器进行安装。例如，在基于 Yum 的系统上：

    ```sql
    $> *yum search libaio*  # search for info
    $> *yum install libaio* # install library
    ```

    或者，在基于 APT 的系统上：

    ```sql
    $> *apt-cache search libaio* # search for info
    $> *apt-get install libaio1* # install library
    ```

+   **Oracle Linux 8 / Red Hat 8**（EL8）：这些平台默认不安装文件 `/lib64/libtinfo.so.5`，而 MySQL 客户端 **bin/mysql** 对于包 `mysql-VERSION-el7-x86_64.tar.gz` 和 `mysql-VERSION-linux-glibc2.12-x86_64.tar.xz` 是必需的。要解决此问题，请安装 `ncurses-compat-libs` 包：

    ```sql
    $> *yum install ncurses-compat-libs*
    ```

要安装压缩的 **tar** 文件二进制发行版，请将其解压缩到您选择的安装位置（通常为 `/usr/local/mysql`）。这将创建以下表中显示的目录。

**表 2.3 通用 Unix/Linux 二进制包 MySQL 安装布局**

| 目录 | 目录内容 |
| --- | --- |
| `bin` | **mysqld** 服务器、客户端和实用程序 |
| `docs` | MySQL 手册的 Info 格式 |
| `man` | Unix 手册页 |
| `include` | 包含（头）文件 |
| `lib` | 库 |
| `share` | 错误消息、字典和数据库安装的 SQL |
| `support-files` | 各种支持文件 |

调试版本的**mysqld**二进制文件可作为**mysqld-debug**使用。要从源代码分发编译自己的 MySQL 调试版本，请使用适当的配置选项以启用调试支持。参见第 2.8 节，“从源代码安装 MySQL”。

要安装和使用 MySQL 二进制发行版，命令序列如下：

```sql
$> groupadd mysql
$> useradd -r -g mysql -s /bin/false mysql
$> cd /usr/local
$> tar xvf */path/to/mysql-VERSION-OS*.tar.xz
$> ln -s *full-path-to-mysql-VERSION-OS* mysql
$> cd mysql
$> mkdir mysql-files
$> chown mysql:mysql mysql-files
$> chmod 750 mysql-files
$> bin/mysqld --initialize --user=mysql
$> bin/mysql_ssl_rsa_setup
$> bin/mysqld_safe --user=mysql &
# Next command is optional
$> cp support-files/mysql.server /etc/init.d/mysql.server
```

注意

此过程假定您对系统具有 `root`（管理员）访问权限。或者，您可以使用 **sudo**（Linux）或 **pfexec**（Solaris）命令为每个命令添加前缀。

`mysql-files` 目录提供了一个方便的位置，可用作 `secure_file_priv` 系统变量的值，该变量将导入和导出操作限制为特定目录。参见第 7.1.8 节，“服务器系统变量”。

安装二进制发行版的更详细版本如下。

### 创建一个 mysql 用户和组

如果您的系统尚未具有用于运行**mysqld**的用户和组，您可能需要创建它们。以下命令添加 `mysql` 组和 `mysql` 用户。您可能希望将用户和组命名为其他名称而不是 `mysql`。如果是这样，请在以下说明中替换适当的名称。在不同版本的 Unix/Linux 上，**useradd** 和 **groupadd** 的语法可能略有不同，或者可能具有不同的名称，如 **adduser** 和 **addgroup**。

```sql
$> groupadd mysql
$> useradd -r -g mysql -s /bin/false mysql
```

注���

因为用户仅用于所有权目的，而非登录目的，所以 **useradd** 命令使用 `-r` 和 `-s /bin/false` 选项创建一个用户，该用户没有登录到服务器主机的权限。如果您的 **useradd** 不支持这些选项，请省略它们。

### 获取并解压发行版

选择您想要解压发行版的目录，并切换到该目录。这里的示例将发行版解压到 `/usr/local`。因此，这些说明假定您有权限在 `/usr/local` 中创建文件和目录。如果该��录受保护，您必须以 `root` 身份执行安装。

```sql
$> cd /usr/local
```

按照 第 2.1.3 节，“如何获取 MySQL” 中的说明获取一个发行版文件。对于特定版本，所有平台的二进制发行版都是从相同的 MySQL 源发行版构建的。

解压发行版，这将创建安装目录。如果 **tar** 支持 `z` 选项，则可以使用 **tar** 来解压缩和解包发行版：

```sql
$> tar xvf */path/to/mysql-VERSION-OS*.tar.xz
```

**tar** 命令创建一个名为 `mysql-*`VERSION`*-*`OS`*` 的目录。

要从压缩的 **tar** 文件二进制发行版安装 MySQL，您的系统必须具有 GNU `XZ Utils` 来解压发行版，并且需要一个合理的 **tar** 来解包它。

注意

MySQL Server 8.0.12 中的压缩算法从 Gzip 更改为 XZ；通用二进制文件的文件扩展名也从 .tar.gz 更改为 .tar.xz。

GNU **tar** 被认为是可用的。一些操作系统提供的标准 **tar** 无法解压 MySQL 发行版中的长文件名。您应该下载并安装 GNU **tar**，或者如果可用，使用预安装的 GNU tar 版本。通常这些版本可以在 GNU 或自由软件目录中找到，比如 `/usr/sfw/bin` 或 `/usr/local/bin` 中的 **gnutar**、**gtar** 或 **tar**。GNU **tar** 可以从 [`www.gnu.org/software/tar/`](http://www.gnu.org/software/tar/) 获取。

如果您的 **tar** 不支持 `xz` 格式，则使用 **xz** 命令解压发行版，然后使用 **tar** 解包。用以下替代命令替换前面的 **tar** 命令以解压缩和提取发行版：

```sql
$> xz -dc */path/to/mysql-VERSION-OS*.tar.xz | tar x
```

接下来，创建一个指向 **tar** 创建的安装目录的符号链接：

```sql
$> ln -s *full-path-to-mysql-VERSION-OS* mysql
```

`ln` 命令创建一个指向安装目录的符号链接。这使您可以更轻松地将其称为 `/usr/local/mysql`。为了避免在使用 MySQL 时总是需要键入客户端程序的路径名，您可以将 `/usr/local/mysql/bin` 目录添加到您的 `PATH` 变量中：

```sql
$> export PATH=$PATH:/usr/local/mysql/bin
```

### 执行后安装设置

安装过程的其余部分涉及设置发行版所有权和访问权限，初始化数据目录，启动 MySQL 服务器，并设置配置文件。有关说明，请参阅 第 2.9 节，“后安装设置和测试”。
