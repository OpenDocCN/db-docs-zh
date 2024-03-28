# 2.7.1 在 Solaris 上使用 Solaris PKG 安装 MySQL

> 原文：[`dev.mysql.com/doc/refman/8.0/en/solaris-installation-pkg.html`](https://dev.mysql.com/doc/refman/8.0/en/solaris-installation-pkg.html)

您可以使用本机 Solaris PKG 格式的二进制软件包安装 MySQL，而不是使用二进制 tarball 分发。

注意

MySQL 5.7 依赖于 Oracle Developer Studio Runtime Libraries；但这不适用于 MySQL 8.0。

要使用此软件包，请下载相应的`mysql-VERSION-solaris11-PLATFORM.pkg.gz`文件，然后解压缩它。例如：

```sql
$> gunzip mysql-*8.0.36*-solaris11-x86_64.pkg.gz
```

要安装新软件包，请使用**pkgadd**并按照屏幕提示操作。您必须具有 root 权限才能执行此���作：

```sql
$> pkgadd -d mysql-*8.0.36*-solaris11-x86_64.pkg

The following packages are available:
  1  mysql     MySQL Community Server (GPL)
               (i86pc) 8.0.36

Select package(s) you wish to process (or 'all' to process
all packages). (default: all) [?,??,q]:
```

PKG 安装程序安装所有所需的文件和工具，然后初始化您的数据库（如果不存在）。要完成安装，您应该根据安装结束时提供的说明设置 MySQL 的 root 密码。或者，您可以运行随安装提供的**mysql_secure_installation**脚本。

默认情况下，PKG 软件包将 MySQL 安装在根路径`/opt/mysql`下。当使用**pkgadd**时，您只能更改安装根路径，该命令可用于在不同的 Solaris 区域中安装 MySQL。如果需要安装在特定目录中，请使用二进制**tar**文件分发。

`pkg`安装程序将适当的 MySQL 启动脚本复制到`/etc/init.d/mysql`中。为了使 MySQL 能够自动启动和关闭，您应该在此文件和 init 脚本目录之间创建链接。例如，为了确保 MySQL 的安全启动和关闭，您可以使用以下命令添加正确的链接：

```sql
$> ln /etc/init.d/mysql /etc/rc3.d/S91mysql
$> ln /etc/init.d/mysql /etc/rc0.d/K02mysql
```

要删除 MySQL，已安装的软件包名称为`mysql`。您可以结合**pkgrm**命令来删除安装。

使用 Solaris 软件包文件格式进行升级时，必须在安装更新软件包之前删除现有安装。卸载软件包不会删除现有的数据库信息，只会删除服务器、二进制文件和支持文件。因此，典型的升级顺序是：

```sql
$> mysqladmin shutdown
$> pkgrm mysql
$> pkgadd -d mysql-*8.0.36*-solaris11-x86_64.pkg
$> mysqld_safe &
$> mysql_upgrade   # prior to MySQL 8.0.16 only
```

在执行任何升级之前，请查看第三章，*升级 MySQL*中的注意事项。
