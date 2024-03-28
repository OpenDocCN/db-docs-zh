> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-deploying-instances.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-deploying-instances.html)

#### 20.2.1.1 部署用于组复制的实例

第一步是部署至少三个 MySQL Server 实例，此过程演示了在多个主机上使用实例的方法，命名为 s1、s2 和 s3。假设每个主机上都安装了 MySQL Server（参见 Chapter 2, *Installing MySQL*）。MySQL Server 8.0 提供了 Group Replication 插件；不需要额外的软件，但插件必须安装在运行中的 MySQL 服务器上。请参见 Section 20.2.1.1, “Deploying Instances for Group Replication”；更多信息，请参见 Section 7.6, “MySQL Server Plugins”。

在这个示例中，使用了三个实例来组成一个组，这是创建组的最小实例数。增加更多实例可以增加组的容错能力。例如，如果组由三个成员组成，在一个实例故障的情况下，组可以继续运行。但在另一个实例故障的情况下，组将无法继续处理写事务。通过增加更多实例，组在继续处理事务的同时可以容忍更多服务器的故障。可以在一个组中使用的最大实例数为九。更多信息请参见 Section 20.1.4.2, “Failure Detection”。
