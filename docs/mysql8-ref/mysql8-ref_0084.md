# 2.8.4 使用标准源分发安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/installing-source-distribution.html`](https://dev.mysql.com/doc/refman/8.0/en/installing-source-distribution.html)

要从标准源分发安装 MySQL：

1.  确保你的系统满足第 2.8.2 节，“安装源的先决条件”中列出的工具要求。

1.  使用第 2.1.3 节，“如何获取 MySQL”中的说明获取分发文件。

1.  使用本节中的说明配置、构建和安装分发。

1.  使用第 2.9 节，“后安装设置和测试”中的说明执行后安装程序。

MySQL 在所有平台上使用**CMake**作为构建框架。这里给出的说明应该能帮助你生成一个可工作的安装。有关使用**CMake**构建 MySQL 的更多信息，请参见使用 CMake 构建 MySQL 服务器。

如果你从源码 RPM 开始，请使用以下命令生成一个二进制 RPM，以便安装。如果你没有**rpmbuild**，请使用**rpm**代替。

```sql
$> rpmbuild --rebuild --clean MySQL-*VERSION*.src.rpm
```

结果是一个或多个二进制 RPM 软件包，你可以按照第 2.5.4 节，“使用 Oracle 的 RPM 软件包在 Linux 上安装 MySQL”中的指示进行安装。

从压缩的**tar**文件或 Zip 存档源分发安装的顺序与从通用二进制分发安装的过程类似（参见第 2.2 节，“在 Unix/Linux 上使用通用二进制安装 MySQL”），只是它适用于所有平台，并包括配置和编译分发的步骤。例如，在 Unix 上使用压缩的**tar**文件源分发，基本安装命令顺序如下：

```sql
# Preconfiguration setup
$> groupadd mysql
$> useradd -r -g mysql -s /bin/false mysql
# Beginning of source-build specific instructions
$> tar zxvf mysql-*VERSION*.tar.gz
$> cd mysql-*VERSION*
$> mkdir bld
$> cd bld
$> cmake ..
$> make
$> make install
# End of source-build specific instructions
# Postinstallation setup
$> cd /usr/local/mysql
$> mkdir mysql-files
$> chown mysql:mysql mysql-files
$> chmod 750 mysql-files
$> bin/mysqld --initialize --user=mysql
$> bin/mysql_ssl_rsa_setup
$> bin/mysqld_safe --user=mysql &
# Next command is optional
$> cp support-files/mysql.server /etc/init.d/mysql.server
```

源码构建特定说明的更详细版本如下所示。

注意

此处显示的过程不设置任何 MySQL 帐户的密码。在按照此过程后，请继续到第 2.9 节，“后安装设置和测试”进行后安装设置和测试。

+   执行预配置设置

+   获取并解压分发

+   配置分发

+   构建分发

+   安装分发

+   执行安装后设置

#### 执行预配置设置

在 Unix 系统上，设置拥有数据库目录并应用于运行和执行 MySQL 服务器的`mysql`用户，以及该用户所属的组。有关详细信息，请参见创建 mysql 用户和组。然后以`mysql`用户的身份执行以下步骤，除非另有说明。

#### 获取并解压缩分发

选择要解压缩分发的目录，并更改位置进入其中。

使用第 2.1.3 节，“获取 MySQL”的说明获取分发文件。

将分发解压缩到当前目录：

+   要解压缩压缩的**tar**文件，如果支持`z`选项，**tar**可以解压缩和解包分发：

    ```sql
    $> tar zxvf mysql-*VERSION*.tar.gz
    ```

    如果您的**tar**不支持`z`选项，请使用**gunzip**解压缩分发，然后使用**tar**解包：

    ```sql
    $> gunzip < mysql-*VERSION*.tar.gz | tar xvf -
    ```

    或者，**CMake**可以解压缩和解包分发：

    ```sql
    $> cmake -E tar zxvf mysql-*VERSION*.tar.gz
    ```

+   要解压缩 Zip 存档，请使用**WinZip**或另一个可以读取`.zip`文件的工具。

解压缩分发文件会创建一个名为`mysql-*`VERSION`*`的目录。

#### 配置分发

将位置更改为解压缩分发的顶级目录：

```sql
$> cd mysql-*VERSION*
```

在源树之外构建以保持树的清洁。如果顶级源目录在您当前的工作目录下名为`mysql-src`，您可以在同一级别的一个名为`build`的目录中构建。创建该目录并进入其中：

```sql
$> mkdir bld
$> cd bld
```

配置构建目录。最小配置命令不包括任何选项以覆盖配置默认值：

```sql
$> cmake ../mysql-src
```

构建目录不需要在源树之外。例如，您可以在顶级源树下的名为`build`的目录中构建。为此，以`mysql-src`作为当前工作目录，创建目录`build`，然后进入该目录：

```sql
$> mkdir build
$> cd build
```

配置构建目录。最小配置命令不包括任何选项以覆盖配置默认值：

```sql
$> cmake ..
```

如果您在同一级别上有多个源树（例如，构建多个 MySQL 版本），第二种策略可能更有优势。第一种策略将所有构建目录放在同一级别，这要求您为每个目录选择一个唯一的名称。使用第二种策略，您可以在每个源树中使用相同的名称来构建目录。以下说明假定采用第二种策略。

在 Windows 上，指定开发环境。例如，以下命令分别为 32 位或 64 位构建配置 MySQL：

```sql
$> cmake .. -G "Visual Studio 12 2013"

$> cmake .. -G "Visual Studio 12 2013 Win64"
```

在 macOS 上，使用 Xcode IDE：

```sql
$> cmake .. -G Xcode
```

当你运行**Cmake**时，可能需要在命令行中添加选项。以下是一些示例：

+   `-DBUILD_CONFIG=mysql_release`: 使用 Oracle 用于生成官方 MySQL 发布的二进制分发的相同构建选项配置源代码。

+   `-DCMAKE_INSTALL_PREFIX=*`dir_name`*`: 配置安装在特定位置下的分发。

+   `-DCPACK_MONOLITHIC_INSTALL=1`: 使**make package**生成单个安装文件而不是多个文件。

+   `-DWITH_DEBUG=1`: 使用调试支持构建分发。

要获取更详尽的选项列表，请参见第 2.8.7 节“MySQL 源配置选项”。

要列出配置选项，请使用以下命令之一：

```sql
$> cmake .. -L   # overview

$> cmake .. -LH  # overview with help text

$> cmake .. -LAH # all params with help text

$> ccmake ..     # interactive display
```

如果**CMake**失败，可能需要使用不同选项再次运行以重新配置。如果重新配置，请注意以下事项：

+   如果**CMake**在之前已经运行过后再次运行，可能会使用在之前调用期间收集的信息。这些信息存储在`CMakeCache.txt`中。当**CMake**启动时，它会查找该文件，并在假设信息仍然正确的情况下读取其内容。当你重新配置时，这种假设是无效的。

+   每次运行**CMake**后，必须再次运行**make**重新编译。但是，你可能需要先删除以前构建时使用不同配置选项编译的旧对象文件。

为了防止旧的对象文件或配置信息被使用，在 Unix 上重新运行**CMake**之前，在构建目录中运行以下命令：

```sql
$> make clean
$> rm CMakeCache.txt
```

或者，在 Windows 上：

```sql
$> devenv MySQL.sln /clean
$> del CMakeCache.txt
```

在向[MySQL 社区 Slack](https://mysqlcommunity.slack.com/)询问之前，请检查`CMakeFiles`目录中的文件，以获取有关失败的有用信息。要提交错误报告，请使用第 1.5 节“如何报告错误或问题”中的说明。

#### 构建分发

在 Unix 上：

```sql
$> make
$> make VERBOSE=1
```

第二个命令设置`VERBOSE`以显示每个编译源的命令。

在使用 GNU **make**并已安装为**gmake**的系统上，请改用**gmake**。

在 Windows 上：

```sql
$> devenv MySQL.sln /build RelWithDebInfo
```

如果您已经到了编译阶段，但分发包无法构建，请参阅第 2.8.8 节，“处理 MySQL 编译问题”以获取帮助。如果这不能解决问题，请按照第 1.5 节，“如何报告错误或问题”中的说明将问题输入到我们的错误数据库中。如果您已安装了所需工具的最新版本，但它们在尝试处理我们的配置文件时崩溃，请也报告这个问题。但是，如果您遇到`command not found`错误或类似的所需工具问题，请不要报告。相反，请确保所有所需工具都已安装，并且您的`PATH`变量设置正确，以便您的 shell 可以找到它们。

#### 安装分发包

在 Unix 上：

```sql
$> make install
```

这将安装文件在配置的安装目录下（默认为`/usr/local/mysql`）。您可能需要以`root`身份运行命令。

要在特定目录中安装，请在命令行中添加一个`DESTDIR`参数：

```sql
$> make install DESTDIR="/opt/mysql"
```

或者，生成安装包文件，您可以在喜欢的位置安装：

```sql
$> make package
```

此操作会生成一个或多个`.tar.gz`文件，可以像通用二进制分发包一样安装。请参阅第 2.2 节，“在 Unix/Linux 上使用通用二进制文件安装 MySQL”。如果使用`-DCPACK_MONOLITHIC_INSTALL=1`运行**CMake**，则该操作会生成一个单一文件。否则，它会生成多个文件。

在 Windows 上，生成数据目录，然后创建一个`.zip`归档安装包：

```sql
$> devenv MySQL.sln /build RelWithDebInfo /project initial_database
$> devenv MySQL.sln /build RelWithDebInfo /project package
```

您可以在喜欢的位置安装生成的`.zip`归档。请参阅第 2.3.4 节，“在 Microsoft Windows 上使用`noinstall` ZIP 归档安装 MySQL”。

#### 执行后安装设置

安装过程的其余部分涉及设置配置文件，创建核心数据库和启动 MySQL 服务器。有关说明，请参阅第 2.9 节，“后安装设置和测试”。

注意

最初列在 MySQL 授权表中的帐户没有密码。启动服务器后，您应该使用第 2.9 节，“后安装设置和测试”中的说明为它们设置密码。
