> 原文：[`dev.mysql.com/doc/refman/8.0/en/thread-pool-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/thread-pool-installation.html)

#### 7.6.3.2 线程池安装

本节描述了如何安装 MySQL Enterprise Thread Pool。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来��置插件目录位置。

插件库文件基本名称为`thread_pool`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为`.so`，对于 Windows 为`.dll`）。

+   MySQL 8.0.14 线程池安装

+   MySQL 8.0.14 之前的线程池安装

##### MySQL 8.0.14 线程池安装

在 MySQL 8.0.14 及更高版本中，线程池监控表是性能模式表，随着线程池插件一起加载和卸载。`INFORMATION_SCHEMA` 版本的表已被弃用，但仍可用；它们按照 MySQL 8.0.14 之前的线程池安装说明中的说明安装。

要启用线程池功能，请通过使用`--plugin-load-add`选项启动服务器加载插件。为此，请将以下行放入服务器的`my.cnf`文件中，并根据需要调整`.so`后缀以适应您的平台：

```sql
[mysqld]
plugin-load-add=thread_pool.so
```

要验证插件安装，请检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见第 7.6.2 节，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'thread%';
+-----------------------+---------------+
| PLUGIN_NAME           | PLUGIN_STATUS |
+-----------------------+---------------+
| thread_pool           | ACTIVE        |
+-----------------------+---------------+
```

要验证性能模式监控表是否可用，请检查信息模式`TABLES`表或使用`SHOW TABLES`语句。例如：

```sql
mysql> SELECT TABLE_NAME
       FROM INFORMATION_SCHEMA.TABLES
       WHERE TABLE_SCHEMA = 'performance_schema'
       AND TABLE_NAME LIKE 'tp%';
+-----------------------+
| TABLE_NAME            |
+-----------------------+
| tp_thread_group_state |
| tp_thread_group_stats |
| tp_thread_state       |
+-----------------------+
```

如果服务器成功加载线程池插件，则将`thread_handling`系统变量设置为`loaded-dynamically`。

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

##### MySQL 8.0.14 之前的线程池安装

在 MySQL 8.0.14 之前，线程池监视表是与线程池插件分开的插件，可以单独安装。

要启用线程池功能，请通过使用`--plugin-load-add`选项启动服务器加载要使用的插件。例如，如果只命名插件库文件，则服务器将加载其中包含的所有插件（即线程池插件和所有`INFORMATION_SCHEMA`表）。为此，请将以下行放入服务器的`my.cnf`文件中，并根据需要调整平台的`.so`后缀：

```sql
[mysqld]
plugin-load-add=thread_pool.so
```

这相当于通过逐个命名加载所有线程池插件：

```sql
[mysqld]
plugin-load-add=thread_pool=thread_pool.so
plugin-load-add=tp_thread_state=thread_pool.so
plugin-load-add=tp_thread_group_state=thread_pool.so
plugin-load-add=tp_thread_group_stats=thread_pool.so
```

如果需要，可以从库文件中加载单独的插件。要加载线程池插件但不加载`INFORMATION_SCHEMA`表，请使用以下选项：

```sql
[mysqld]
plugin-load-add=thread_pool=thread_pool.so
```

要加载线程池插件和仅`TP_THREAD_STATE` `INFORMATION_SCHEMA`表，请使用以下选项：

```sql
[mysqld]
plugin-load-add=thread_pool=thread_pool.so
plugin-load-add=tp_thread_state=thread_pool.so
```

要验证插件安装，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见第 7.6.2 节，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'thread%' OR PLUGIN_NAME LIKE 'tp%';
+-----------------------+---------------+
| PLUGIN_NAME           | PLUGIN_STATUS |
+-----------------------+---------------+
| thread_pool           | ACTIVE        |
| TP_THREAD_STATE       | ACTIVE        |
| TP_THREAD_GROUP_STATE | ACTIVE        |
| TP_THREAD_GROUP_STATS | ACTIVE        |
+-----------------------+---------------+
```

如果服务器成功加载线程池插件，则将`thread_handling`系统变量设置为`loaded-dynamically`。

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。
