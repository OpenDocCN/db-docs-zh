> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-service.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-service.html)

#### 25.3.2.4 将 NDB Cluster 进程安装为 Windows 服务

一旦您确认 NDB Cluster 运行如您所期望，您可以将管理节点和数据节点安装为 Windows 服务，这样这些进程将在 Windows 启动或停止时自动启动和停止。这也使得可以使用适当的**SC START**和**SC STOP**命令，或使用 Windows 图形**Services**实用程序来控制这些进程。也可以使用**NET START**和**NET STOP**命令。

通常必须使用具有系统管理员权限的帐户来将程序安装为 Windows 服务。

要在 Windows 上将管理节点安装为服务，请在托管管理节点的计算机上使用命令行调用**ndb_mgmd.exe**，使用`--install`选项，如下所示：

```sql
C:\> C:\mysql\bin\ndb_mgmd.exe --install
Installing service 'NDB Cluster Management Server'
  as '"C:\mysql\bin\ndbd.exe" "--service=ndb_mgmd"'
Service successfully installed.
```

重要

在将 NDB Cluster 程序安装为 Windows 服务时，应始终指定完整路径；否则服务安装可能会因为错误“系统找不到指定的文件”而失败。

必须首先使用`--install`选项，然后再指定可能用于**ndb_mgmd.exe**的任何其他选项。然而，最好是在选项文件中指定这些选项。如果您的选项文件不在**ndb_mgmd.exe** `--help`输出中显示的默认位置之一，您可以使用`--config-file`选项指定位置。

现在您应该能够像这样启动和停止管理服务器：

```sql
C:\> SC START ndb_mgmd

C:\> SC STOP ndb_mgmd
```

注意

如果使用**NET**命令，您也可以使用描述性名称启动或停止管理服务器作为 Windows 服务，如下所示：

```sql
C:\> NET START 'NDB Cluster Management Server'
The NDB Cluster Management Server service is starting.
The NDB Cluster Management Server service was started successfully.

C:\> NET STOP  'NDB Cluster Management Server'
The NDB Cluster Management Server service is stopping..
The NDB Cluster Management Server service was stopped successfully.
```

通常在安装服务时指定一个简短的服务名称或允许使用默认服务名称，然后在启动或停止服务时引用该名称会更简单。要指定一个不同于`ndb_mgmd`的服务名称，请将其附加到`--install`选项中，如下例所示：

```sql
C:\> C:\mysql\bin\ndb_mgmd.exe --install=mgmd1
Installing service 'NDB Cluster Management Server'
  as '"C:\mysql\bin\ndb_mgmd.exe" "--service=mgmd1"'
Service successfully installed.
```

现在您应该能够使用您指定的名称启动或停止服务，就像这样：

```sql
C:\> SC START mgmd1

C:\> SC STOP mgmd1
```

要删除管理节点服务，请使用**SC DELETE *`service_name`***：

```sql
C:\> SC DELETE mgmd1
```

或者，使用**ndb_mgmd.exe**并带上`--remove`选项，如下所示：

```sql
C:\> C:\mysql\bin\ndb_mgmd.exe --remove
Removing service 'NDB Cluster Management Server'
Service successfully removed.
```

如果使用非默认服务名称安装服务，请将服务名称作为**ndb_mgmd.exe** `--remove`选项的值传递，如下所示：

```sql
C:\> C:\mysql\bin\ndb_mgmd.exe --remove=mgmd1
Removing service 'mgmd1'
Service successfully removed.
```

将 NDB Cluster 数据节点进程安装为 Windows 服务的操作方式类似，使用**ndbd.exe**的`--install`选项（或**ndbmtd.exe**"))，如下所示：

```sql
C:\> C:\mysql\bin\ndbd.exe --install
Installing service 'NDB Cluster Data Node Daemon' as '"C:\mysql\bin\ndbd.exe" "--service=ndbd"'
Service successfully installed.
```

现在您可以按照以下示例启动或停止数据节点：

```sql
C:\> SC START ndbd

C:\> SC STOP ndbd
```

要删除数据节点服务，请使用**SC DELETE *`service_name`***：

```sql
C:\> SC DELETE ndbd
```

或者，使用**ndbd.exe**并带上`--remove`选项，如下所示：

```sql
C:\> C:\mysql\bin\ndbd.exe --remove
Removing service 'NDB Cluster Data Node Daemon'
Service successfully removed.
```

与**ndb_mgmd.exe**（和**mysqld.exe**)一样，当将**ndbd.exe**安装为 Windows 服务时，也可以指定服务名称作为`--install`的值，然后在启动或停止服务时使用它，如下所示：

```sql
C:\> C:\mysql\bin\ndbd.exe --install=dnode1
Installing service 'dnode1' as '"C:\mysql\bin\ndbd.exe" "--service=dnode1"'
Service successfully installed.

C:\> SC START dnode1

C:\> SC STOP dnode1
```

如果在安装数据节点服务时指定了服务名称，那么在删除时也可以使用该名称，如下所示：

```sql
C:\> SC DELETE dnode1
```

或者，可以将服务名称作为`ndbd.exe` `--remove`选项的值传递，如下所示：

```sql
C:\> C:\mysql\bin\ndbd.exe --remove=dnode1
Removing service 'dnode1'
Service successfully removed.
```

将 SQL 节点安装为 Windows 服务，启动服务，停止服务和删除服务的操作方式类似，使用**mysqld** `--install`，**SC START**，**SC STOP**和**SC DELETE**（或**mysqld** `--remove`）。也可以使用**NET**命令来启动或停止服务。有关更多信息，请参见 Section 2.3.4.8, “Starting MySQL as a Windows Service”。
