> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-semisync-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-semisync-installation.html)

#### 19.4.10.1 安装半同步复制

半同步复制是使用插件实现的，这些插件必须安装在源服务器和副本上，以使半同步复制在实例上可用。 源和副本有不同的插件。 安装插件后，您可以通过与之关联的系统变量来控制它。 仅当关联的插件已安装时，这些系统变量才可用。

本节描述了如何安装半同步复制插件。 有关安装插件的一般信息，请参见 Section 7.6.1, “安装和卸载插件”。

要使用半同步复制，必须满足以下要求：

+   安装插件的能力需要支持动态加载的 MySQL 服务器。 要验证这一点，请检查`have_dynamic_loading` 系统变量的值是否为 `YES`。 二进制发行版应支持动态加载。

+   复制必须已经正常工作，请参见 Section 19.1, “配置复制”。

+   不得配置多个复制通道。 半同步复制仅与默认复制通道兼容。 请参见 Section 19.2.2, “复制通道”。

从 MySQL 8.0.26 开始，提供了实现半同步复制的插件的新版本，一个用于源服务器，一个用于副本。 新插件在系统变量和状态变量中将“主”和“从”替换为“源”和“副本”，您可以安装这些版本而不是旧版本。 不能在实例上同时安装相关插件的新旧版本。 如果使用新版本的插件，则新的系统变量和状态变量可用，但旧的不可用。 如果使用旧版本的插件，则旧的系统变量和状态变量可用，但新的不可用。

插件库文件的文件名后缀因平台而异（例如，Unix 和类 Unix 系统使用`.so`，Windows 使用`.dll`）。 插件和库文件的名称如下：

+   源服务器，旧术语：`rpl_semi_sync_master` 插件（`semisync_master.so` 或 `semisync_master.dll` 库）

+   源服务器，新术语（从 MySQL 8.0.26 开始）：`rpl_semi_sync_source` 插件（`semisync_source.so` 或 `semisync_source.dll` 库）

+   复制品，旧术语：`rpl_semi_sync_slave` 插件（`semisync_slave.so` 或 `semisync_slave.dll` 库）

+   复制，新术语（自 MySQL 8.0.26 起）：`rpl_semi_sync_replica`插件（`semisync_replica.so`或`semisync_replica.dll`库）

要被源服务器或复制服务器使用，适当的插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。必要时，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。源插件库文件必须存在于源服务器的插件目录中。复制插件库文件必须存在于每个复制服务器的插件目录中。

要设置半同步复制，请使用以下说明。这里提到的`INSTALL PLUGIN`、`SET GLOBAL`、`STOP REPLICA`和`START REPLICA`语句需要`REPLICATION_SLAVE_ADMIN`权限（或已弃用的`SUPER`权限）。

加载插件，请在源端和每个要进行半同步的复制端上使用`INSTALL PLUGIN`语句，根据需要调整您平台的`.so`后缀。

在源端：

```sql
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';

Or from MySQL 8.0.26:
INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
```

在每个复制端：

```sql
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

Or from MySQL 8.0.26:
INSTALL PLUGIN rpl_semi_sync_replica SONAME 'semisync_replica.so';
```

如果在 Linux 上尝试安装插件时出现类似于此处所示的错误，则必须安装`libimf`：

```sql
mysql> INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
ERROR 1126 (HY000): Can't open shared library
'/usr/local/mysql/lib/plugin/semisync_source.so'
(errno: 22 libimf.so: cannot open shared object file:
No such file or directory)
```

您可以从[`dev.mysql.com/downloads/os-linux.html`](https://dev.mysql.com/downloads/os-linux.html)获取`libimf`。

要验证插件安装，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见第 7.6.2 节，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE '%semi%';
+----------------------+---------------+
| PLUGIN_NAME          | PLUGIN_STATUS |
+----------------------+---------------+
| rpl_semi_sync_source | ACTIVE        |
+----------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

安装半同步复制插件后，默认情况下是禁用的。必须在源端和复制端都启用插件才能启用半同步复制。如果只有一侧启用，则复制是异步的。要启用插件，请在运行时使用`SET GLOBAL`设置适当的系统变量，或在命令行或选项文件中在服务器启动时设置。例如：

```sql
On the source:
SET GLOBAL rpl_semi_sync_master_enabled = 1;

Or from MySQL 8.0.26 with the rpl_semi_sync_source plugin:
SET GLOBAL rpl_semi_sync_source_enabled = 1;
```

```sql
On each replica:
SET GLOBAL rpl_semi_sync_slave_enabled = 1;

Or from MySQL 8.0.26 with the rpl_semi_sync_replica plugin:
SET GLOBAL rpl_semi_sync_replica_enabled = 1;
```

如果您在运行时在副本上启用半同步复制，您还必须启动复制 I/O（接收器）线程（如果已经运行，则首先停止它），以使副本连接到源并注册为半同步副本：

```sql
STOP SLAVE IO_THREAD;
START SLAVE IO_THREAD;

Or from MySQL 8.0.22:
STOP REPLICA IO_THREAD;
START REPLICA IO_THREAD;
```

如果复制 I/O（接收器）线程已经在运行，而您没有重新启动它，副本将继续使用异步复制。

在选项文件中列出的设置在每次服务器启动时生效。例如，您可以在源服务器和副本服务器的`my.cnf`文件中设置变量如下：

```sql
 On the source: 

[mysqld]
rpl_semi_sync_master_enabled=1

Or from MySQL 8.0.26 with the rpl_semi_sync_source plugin:
rpl_semi_sync_source_enabled=1
```

```sql
 On each replica: 

[mysqld]
rpl_semi_sync_slave_enabled=1

Or from MySQL 8.0.26 with the rpl_semi_sync_source plugin:
rpl_semi_sync_replica_enabled=1
```

您可以使用安装插件时提供的系统变量来配置半同步复制插件的行为。有关关键系统变量的信息，请参见 Section 19.4.10.2, “Configuring Semisynchronous Replication”。
