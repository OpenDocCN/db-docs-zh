# 20.2.2 本地部署组复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-deploying-locally.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-deploying-locally.html)

部署组复制最常见的方式是使用多个服务器实例，以提供高可用性。也可以在本地部署组复制，例如用于测试目的。本节解释了如何在本地部署组复制。

重要

组复制通常部署在多个主机上，因为这样可以提供高可用性。本节中的说明不适用于生产部署，因为所有 MySQL 服务器实例都在同一台主机上运行。如果此主机发生故障，整个组都会失败。因此，此信息应仅用于测试目的，不应在生产环境中使用。

本节解释了如何在一台物理机器上创建一个包含三个 MySQL 服务器实例的复制组。这意味着需要三个数据目录，每个服务器实例一个，并且需要独立配置每个实例。此过程假定 MySQL 服务器已下载并解压缩到名为`mysql-8.0`的目录中。每个 MySQL 服务器实例都需要一个特定的数据目录。创建一个名为`data`的目录，然后在该目录中为每个服务器实例创建一个子目录，例如`s1`、`s2`和`s3`，并对每个实例进行初始化。

```sql
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s1
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s2
mysql-8.0/bin/mysqld --initialize-insecure --basedir=$PWD/mysql-8.0 --datadir=$PWD/data/s3
```

在`data/s1`、`data/s2`、`data/s3`中是一个已初始化的数据目录，包含 mysql 系统数据库和相关表等等。要了解更多关于初始化过程的信息，请参见第 2.9.1 节，“初始化数据目录”。

警告

在生产环境中不要使用`-initialize-insecure`，这只是为了简化教程而使用的。有关安全设置的更多信息，请参见第 20.6 节，“组复制安全”。

#### 本地组复制成员的配置

当您按照第 20.2.1.2 节，“为组复制配置实例”时，您需要为前一节添加的数据目录添加配置。例如：

```sql
[mysqld]

# server configuration
datadir=<full_path_to_data>/data/s1
basedir=<full_path_to_bin>/mysql-8.0/

port=24801
socket=<full_path_to_sock_dir>/s1.sock
```

这些设置配置了 MySQL 服务器使用之前创建的数据目录以及服务器应该打开和开始监听传入连接的端口。

注意

本教程中使用非默认端口 24801，因为三个服务器实例使用相同的主机名。在三台不同机器的设置中，这是不需要的。

组复制需要成员之间的网络连接，这意味着每个成员必须能够解析所有其他成员的网络地址。例如，在本教程中，所有三个实例都在一台机器上运行，因此为了确保成员之间可以联系，您可以在选项文件中添加一行，例如`report_host=127.0.0.1`。

然后，每个成员需要能够连接到其他成员的`group_replication_local_address`。例如，在成员 s1 的选项文件中添加：

```sql
group_replication_local_address= "127.0.0.1:24901"
group_replication_group_seeds= "127.0.0.1:24901,127.0.0.1:24902,127.0.0.1:24903"
```

这将配置 s1 使用端口 24901 进行与种子成员的内部组通信。对于要添加到组中的每个服务器实例，请在成员的选项文件中进行这些更改。对于每个成员，您必须确保指定了唯一地址，因此对于`group_replication_local_address`，每个实例使用唯一端口。通常，您希望所有成员都能够作为加入组并且尚未通过组处理事务的成员的种子，因此，请像上面显示的那样将所有端口添加到`group_replication_group_seeds`。

第 20.2.1 节，“在单主模式下部署组复制”的其余步骤同样适用于您以这种方式在本地部署的组。
