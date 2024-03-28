# 7.6.1 安装和卸载插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/plugin-loading.html`](https://dev.mysql.com/doc/refman/8.0/en/plugin-loading.html)

必须在服务器中加载服务器插件才能使用它们。MySQL 支持在服务器启动和运行时加载插件。还可以在启动时控制已加载插件的激活状态，并在运行时卸载它们。

插件加载后，可以根据第 7.6.2 节“获取服务器插件信息”中的描述获取有关插件的信息。

+   安装插件

+   控制插件激活状态

+   卸载插件

+   插件和可加载函数

#### 安装插件

在使用服务器插件之前，必须使用以下方法之一安装它。在描述中，*`plugin_name`*代表插件名称，如`innodb`、`csv`或`validate_password`。 

+   内置插件

+   注册在 mysql.plugin 系统表中的插件

+   使用命令行选项命名的插件

+   使用 INSTALL PLUGIN 语句安装的插件

##### 内置插件

服务器会自动识别内置插件。默认情况下，服务器在启动时启用插件。一些内置插件允许使用`--*`plugin_name`*[=*`activation_state`*]`选项进行更改。

##### 注册在 mysql.plugin 系统表中的插件

`mysql.plugin`系统表用作插件的注册表（内置插件无需注册）。在正常启动序列期间，服务器会加载在表中注册的插件。默认情况下，对于从`mysql.plugin`表加载的插件，服务器还会启用插件。这可以通过`--*`plugin_name`*[=*`activation_state`*]`选项进行更改。

如果服务器使用`--skip-grant-tables`选项启动，`mysql.plugin`表中注册的插件将不会被加载，也将无法使用。

##### 使用命令行选项命名的插件

位于插件库文件中的插件可以通过`--plugin-load`、`--plugin-load-add`或`--early-plugin-load`选项在服务器启动时加载。通常，对于在启动时加载的插件，服务器还会启用该插件。这可以通过`--*`plugin_name`*[=*`activation_state`*]`选项进行更改。

`--plugin-load`和`--plugin-load-add`选项在服务器启动序列期间内置插件和存储引擎初始化后加载插件。`--early-plugin-load`选项用于加载必须在内置插件和存储引擎初始化之前可用的插件。

每个插件加载选项的值是一个以分号分隔的*`plugin_library`*和*`name`*`=`*`plugin_library`*值的列表。每个*`plugin_library`*是包含插件代码的库文件的名称，每个*`name`*是要加载的插件的名称。如果插件库的名称没有任何前置插件名称，服务器将加载库中的所有插件。有了前置插件名称，服务器将仅从库中加载指定的插件。服务器在由`plugin_dir`系统变量命名的目录中查找插件库文件。

插件加载选项不会在`mysql.plugin`表中注册任何插件。对于后续的重新启动，只有在再次提供`--plugin-load`、`--plugin-load-add`或`--early-plugin-load`时，服务器才会再次加载插件。也就是说，该选项产生一次性的插件安装操作，仅持续一个服务器调用。

`--plugin-load`、`--plugin-load-add`和`--early-plugin-load`使得即使在给定`--skip-grant-tables`的情况下（导致服务器忽略`mysql.plugin`表），也能加载插件。`--plugin-load`、`--plugin-load-add`和`--early-plugin-load`还使得在启动时加载无法在运行时加载的插件成为可能。

`--plugin-load-add`选项是对`--plugin-load`选项的补充：

+   每个`--plugin-load`的实例都会在启动时重置要加载的插件集合，而`--plugin-load-add`会向要加载的插件集合添加一个或多个插件，而不会重置当前集合。因此，如果指定了多个`--plugin-load`的实例，只有最后一个会生效。对于多个`--plugin-load-add`的实例，所有实例都会生效。

+   参数格式与`--plugin-load`相同，但可以使用多个`--plugin-load-add`的实例来避免将大量插件作为单个长而难以控制的`--plugin-load`参数进行指定。

+   可以在没有`--plugin-load`的情况下给出`--plugin-load-add`，但是在`--plugin-load`之前出现的任何`--plugin-load-add`实例都不会生效，因为`--plugin-load`会重置要加载的插件集合。

例如，这些选项：

```sql
--plugin-load=x --plugin-load-add=y
```

等同于这些选项：

```sql
--plugin-load-add=x --plugin-load-add=y
```

并且等同于这个选项：

```sql
--plugin-load="x;y"
```

但这些选项：

```sql
--plugin-load-add=y --plugin-load=x
```

等同于这个选项：

```sql
--plugin-load=x
```

##### 使用 INSTALL PLUGIN 语句安装的插件

位于插件库文件中的插件可以通过`INSTALL PLUGIN`语句在运行时加载。该语句还会在`mysql.plugin`表中注册插件，以便导致服务器在后续重新启动时加载它。因此，`INSTALL PLUGIN`需要对`mysql.plugin`表的`INSERT`权限。

插件库文件的基本名称取决于您的平台。Unix 和类 Unix 系统通常使用`.so`作为后缀，Windows 则使用`.dll`。

示例：`--plugin-load-add`选项在服务器启动时安装插件。要从名为`somepluglib.so`的插件库文件中安装名为`myplugin`的插件，请在`my.cnf`文件中使用以下行：

```sql
[mysqld]
plugin-load-add=myplugin=somepluglib.so
```

在这种情况下，插件不会在`mysql.plugin`中注册。在没有`--plugin-load-add`选项的情况下重新启动服务器会导致插件在启动时不被加载。

替代地，`INSTALL PLUGIN`语句会导致服务器在运行时从库文件加载插件代码：

```sql
INSTALL PLUGIN myplugin SONAME 'somepluglib.so';
```

`INSTALL PLUGIN`还会导致“永久”插件注册：插件将列在`mysql.plugin`表中，以确保服务器在后续重新启动时加载它。

许多插件可以在服务器启动时或运行时加载。但是，如果插件设计为必须在服务器启动期间加载和初始化，则尝试使用`INSTALL PLUGIN`在运行时加载插件会产生错误：

```sql
mysql> INSTALL PLUGIN myplugin SONAME 'somepluglib.so';
ERROR 1721 (HY000): Plugin 'myplugin' is marked as not dynamically
installable. You have to stop the server to install it.
```

在这种情况下，必须使用`--plugin-load`、`--plugin-load-add`或`--early-plugin-load`。

如果插件既使用`--plugin-load`、`--plugin-load-add`或`--early-plugin-load`选项命名，又（由于先前的`INSTALL PLUGIN`语句的结果）在`mysql.plugin`表中，服务器会启动，但会将这些消息写入错误日志：

```sql
[ERROR] Function '*plugin_name*' already exists
[Warning] Couldn't load plugin named '*plugin_name*'
with soname '*plugin_object_file*'.
```

#### 控制插件激活状态

如果服务器在启动时知道插件（例如，因为插件是使用`--plugin-load-add`选项命名或在`mysql.plugin`表中注册的），服务器会默认加载和启用插件。可以使用`--*`plugin_name`*[=*`activation_state`*]`启动选项来控制此类插件的激活状态，其中*`plugin_name`*是要影响的插件名称，如`innodb`、`csv`或`validate_password`。与其他选项一样，选项名称中的破折号和下划线是可以互换的。此外，激活状态值不区分大小写。例如，`--my_plugin=ON`和`--my-plugin=on`是等效的。

+   `--*`plugin_name`*=OFF`

    告诉服务器禁用插件。对于某些内置插件，如`mysql_native_password`，可能无法实现。

+   `--*`plugin_name`*[=ON]`

    告诉服务器启用插件。（不带值地指定选项为`--*`plugin_name`*`具有相同效果。）如果插件初始化失败，服务器将以插件禁用状态运行。

+   `--*`plugin_name`*=FORCE`

    告诉服务器启用插件，但如果插件初始化失败，服务器将不会启动。换句话说，此选项强制服务器以插件启用或完全禁用的方式运行。

+   `--*`plugin_name`*=FORCE_PLUS_PERMANENT`

    类似于`FORCE`，但另外防止插件在运行时卸载。如果用户尝试使用`UNINSTALL PLUGIN`这样做，将会出现错误。

插件激活状态可在信息模式`PLUGINS`表的`LOAD_OPTION`列中看到。

假设`CSV`、`BLACKHOLE`和`ARCHIVE`是内置的可插拔存储引擎，并且您希望服务器在启动时加载它们，但要满足以下条件：如果`CSV`初始化失败，服务器允许运行，必须要求`BLACKHOLE`初始化成功，并且应禁用`ARCHIVE`。为了实现这一点，在选项文件中使用以下行：

```sql
[mysqld]
csv=ON
blackhole=FORCE
archive=OFF
```

`--enable-*`plugin_name`*`选项格式是`--*`plugin_name`*=ON`的同义词。`--disable-*`plugin_name`*`和`--skip-*`plugin_name`*`选项格式是`--*`plugin_name`*=OFF`的同义词。

如果插件被禁用，要么明确使用`OFF`禁用，要么因为启用了`ON`但初始化失败而隐式禁用，那么需要插件的服务器操作方面发生变化。例如，如果插件实现了存储引擎，那么现有的存储引擎表将变得无法访问，并且尝试为存储引擎创建新表将导致使用默认存储引擎的表，除非启用了`NO_ENGINE_SUBSTITUTION` SQL 模式以导致发生错误。

禁用插件可能需要调整其他选项。例如，如果您使用`--skip-innodb`启动服务器以禁用`InnoDB`，则启动时可能还需要省略其他`innodb_*`xxx`*`选项。此外，因为`InnoDB`是默认存储引擎，除非您使用`--default_storage_engine`指定另一个可用的存储引擎，否则它无法启动。您还必须设置`--default_tmp_storage_engine`。

#### 卸载插件

在运行时，`UNINSTALL PLUGIN`语句会禁用并卸载服务器已知的插件。该语句会卸载插件并从`mysql.plugin`系统表中删除它（如果在那里注册）。因此，`UNINSTALL PLUGIN`语句需要对`mysql.plugin`表具有`DELETE`权限。由于插件不再在表中注册，服务器在后续重新启动期间不会加载插件。

`UNINSTALL PLUGIN`可以卸载插件，无论它是在运行时使用`INSTALL PLUGIN`加载还是在启动时使用插件加载选项加载，但要满足以下条件：

+   无法卸载内置于服务器中的插件。这些可以通过信息模式`PLUGINS`表或`SHOW PLUGINS`的输出中具有`NULL`库名称的插件来识别。

+   无法卸载在服务器启动时使用`--*`plugin_name`*=FORCE_PLUS_PERMANENT`启动的插件，这会阻止运行时插件的卸载。这些可以从`PLUGINS`表的`LOAD_OPTION`列中识别出来。

要卸载当前在服务器启动时使用插件加载选项加载的插件，请使用以下步骤。

1.  从`my.cnf`文件中删除与插件相关的任何选项和系统变量。如果任何插件系统变量已经持久化到`mysqld-auto.cnf`文件中，请使用`RESET PERSIST *`var_name`*`来逐个删除它们。

1.  重新启动服务器。

1.  通常情况下，插件可以通过启动时的插件加载选项或在运行时使用`INSTALL PLUGIN`来安装，但不能同时使用两者。然而，如果在某个时刻还使用了`INSTALL PLUGIN`，那么仅仅从`my.cnf`文件中删除插件的选项可能不足以卸载它。如果插件仍然出现在`PLUGINS`或`SHOW PLUGINS`的输出中，请使用`UNINSTALL PLUGIN`将其从`mysql.plugin`表中移除。然后再次重启服务器。

#### 插件和可加载函数

安装插件时，可能还会自动安装相关的可加载函数。如果是这样，那么卸载插件时也会自动卸载这些函数。
