> 原文：[`dev.mysql.com/doc/refman/8.0/en/install-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/install-plugin.html)

#### 15.7.4.4 安装插件语句

```sql
INSTALL PLUGIN *plugin_name* SONAME '*shared_library_name*'
```

此语句安装服务器插件。它需要对`mysql.plugin`系统表的`INSERT`权限，因为它向该表添加一行以注册插件。

*`plugin_name`* 是插件的名称，定义在库文件中包含的插件描述符结构中（参见插件数据结构）。插件名称不区分大小写。为了最大兼容性，插件名称应该限制为 ASCII 字母、数字和下划线，因为它们在 C 源文件、shell 命令行、M4 和 Bourne shell 脚本以及 SQL 环境中使用。

*`shared_library_name`* 是包含插件代码的共享库的名称。该名称包括文件名扩展名（例如，`libmyplugin.so`，`libmyplugin.dll`，或 `libmyplugin.dylib`）。

共享库必须位于插件目录中（由`plugin_dir`系统变量命名的目录）。库必须位于插件目录本身，而不是子目录中。默认情况下，`plugin_dir`是由`pkglibdir`配置变量命名的目录下的`plugin`目录，但可以通过在服务器启动时设置`plugin_dir`的值来更改。例如，在`my.cnf`文件中设置其值：

```sql
[mysqld]
plugin_dir=*/path/to/plugin/directory*
```

如果`plugin_dir`的值是相对路径名，则被视为相对于 MySQL 基本目录（`basedir`系统变量的值）。

`INSTALL PLUGIN` 加载并初始化插件代码，使插件可供使用。插件通过执行其初始化函数进行初始化，该函数处理插件在可以使用之前必须执行的任何设置。当服务器关闭时，它会执行每个已加载插件的去初始化函数，以便插件有机会执行任何最终清理。

`INSTALL PLUGIN`还通过向`mysql.plugin`系统表添加指示插件名称和库文件名的行来注册插件。在正常启动序列期间，服务器加载和初始化在`mysql.plugin`中注册的插件。这意味着插件仅通过`INSTALL PLUGIN`安装一次，而不是每次服务器启动时都安装。如果使用`--skip-grant-tables`选项启动服务器，则在`mysql.plugin`表中注册的插件不会被加载，也无法使用。

插件库可以包含多个插件。为了安装每个插件，使用单独的`INSTALL PLUGIN`语句。每个语句命名不同的插件，但它们都指定相同的库名称。

`INSTALL PLUGIN`会导致服务器在启动时读取选项（`my.cnf`）文件，使得插件可以从这些文件中获取任何相关选项。甚至可以在加载插件之前将插件选项添加到选项文件中（如果使用`loose`前缀）。也可以卸载插件，编辑`my.cnf`，然后再次安装插件。通过这种方式重新启动插件，使其能够在无需重新启动服务器的情况下使用新的选项值。

对于控制单个插件在服务器启动时加载的选项，请参阅第 7.6.1 节，“安装和卸载插件”。如果需要在给定`--skip-grant-tables`选项（告诉服务器不要读取系统表）的情况下为单个服务器启动加载插件，请使用`--plugin-load`选项。请参阅第 7.1.7 节，“服务器命令选项”。

要移除插件，请使用`UNINSTALL PLUGIN`语句。

有关插件加载的其他信息，请参阅第 7.6.1 节，“安装和卸载插件”。

要查看已安装的插件，请使用`SHOW PLUGINS`语句或查询`INFORMATION_SCHEMA`的`PLUGINS`表。

如果重新编译插件库并需要重新安装它，可以使用以下任一方法：

+   使用`UNINSTALL PLUGIN`命令卸载库中的所有插件，将新的插件库文件安装到插件目录中，然后使用`INSTALL PLUGIN`命令安装库中的所有插件。这个过程的优点是可以在不停止服务器的情况下使用。然而，如果插件库包含许多插件，您必须发出许多`INSTALL PLUGIN`和`UNINSTALL PLUGIN`命令。

+   停止服务器，将新的插件库文件安装到插件目录中，然后重新启动服务器。
