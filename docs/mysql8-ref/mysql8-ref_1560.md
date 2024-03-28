> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-upgrade-with-mysqlbackup.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-upgrade-with-mysqlbackup.html)

#### 20.8.3.4 使用**mysqlbackup**进行组复制升级

作为一种供应方法的一部分，您可以使用 MySQL 企业备份将数据从一个组成员复制并恢复到新成员。但是，您不能直接使用此技术将从运行较旧版本 MySQL 的成员中获取的备份直接恢复到运行较新版本 MySQL 的成员。解决方案是将备份恢复到运行与备份来源成员相同版本 MySQL 的新服务器实例，然后升级该实例。此过程包括：

+   从较旧组的成员使用**mysqlbackup**进行备份。参见第 20.5.6 节，“使用 MySQL 企业备份与组复制”。

+   部署一个新的服务器实例，该实例必须运行与获取备份的较旧成员相同版本的 MySQL。

+   使用**mysqlbackup**将备份从较旧成员恢复到新实例。

+   在新实例上升级 MySQL，请参见第三章，*升级 MySQL*。

重复此过程以创建适当数量的新实例，例如以处理故障转移。然后根据第 20.8.3.3 节，“组复制在线升级方法”将实例加入到组中。
