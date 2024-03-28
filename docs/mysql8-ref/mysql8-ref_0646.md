# 10.14.8 NDB 集群线程状态

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-thread-states.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-thread-states.html)

+   `将事件提交到 binlog`

+   `打开 mysql.ndb_apply_status`

+   `处理事件`

    该线程正在处理用于二进制日志记录的事件。

+   `处理模式表中的事件`

    该线程正在进行模式复制的工作。

+   `正在关闭`

+   `同步 ndb 表模式操作和 binlog`

    这用于对 NDB 的模式操作进行正确的二进制日志记录。

+   `等待允许获取 ndbcluster 全局模式锁`

    该线程正在等待获取全局模式锁的许可。

+   `等待来自 ndbcluster 的事件`

    服务器作为 NDB 集群中的 SQL 节点，连接到集群管理节点。

+   `等待来自 ndbcluster 的第一个事件`

+   `等待 ndbcluster 二进制日志更新到当前位置`

+   `等待 ndbcluster 全局模式锁`

    该线程正在等待另一个线程持有的全局模式锁被释放。

+   `等待 ndbcluster 启动`

+   `等待模式时代`

    该线程正在等待模式时代（即全局检查点）。
