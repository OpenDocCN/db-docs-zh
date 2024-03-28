# 25.4 NDB Cluster 的配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-configuration.html)

25.4.1 NDB Cluster 的快速测试设置

25.4.2 NDB Cluster 配置参数、选项和变量概述

25.4.3 NDB Cluster 配置文件

25.4.4 使用高速互连与 NDB Cluster

作为 NDB Cluster 的一部分的 MySQL 服务器在一个主要方面与普通（非集群）MySQL 服务器不同，即它使用`NDB`存储引擎。这个引擎有时也被称为`NDBCLUSTER`，尽管更倾向于使用`NDB`。

为了避免不必要的资源分配，默认情况下配置服务器时会禁用`NDB`存储引擎。要启用`NDB`，必须修改服务器的`my.cnf`配置文件，或使用`--ndbcluster`选项启动服务器。

这个 MySQL 服务器是集群的一部分，因此它还必须知道如何访问管理节点以获取集群配置数据。默认行为是在`localhost`上查找管理节点。但是，如果您需要指定其位置在其他地方，可以在`my.cnf`中或使用**mysql**客户端来完成。在使用`NDB`存储引擎之前，至少必须有一个管理节点运行，以及任何所需的数据节点。

有关`--ndbcluster`和其他特定于 NDB Cluster 的**mysqld**选项的更多信息，请参见 Section 25.4.3.9.1，“NDB Cluster 的 MySQL 服务器选项”。

有关安装 NDB Cluster 的一般信息，请参见 Section 25.3，“NDB Cluster 安装”。
