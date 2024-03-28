> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-error-messages.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-error-messages.html)

#### 25.6.16.33 ndbinfo error_messages 表

`error_messages` 表提供有关

`error_messages` 表包含以下列：

+   `错误代码`

    数字错误代码

+   `错误描述`

    错误描述

+   `error_status`

    错误状态码

+   `错误分类`

    错误分类代码

##### 注意

`error_code` 是一个数字 NDB 错误代码。这是可以提供给 **ndb_perror** 的相同错误代码。

`错误描述` 提供导致错误的条件的基本描述。

`error_status` 列提供与错误相关的状态信息。此列的可能值在此处列出：

+   `无错误`

+   `非法连接字符串`

+   `非法服务器句柄`

+   `服务器回复非法`

+   `节点数量非法`

+   `非法节点状态`

+   `内存不足`

+   `管理服务器未连接`

+   `无法连接到套接字`

+   `启动失败`

+   `停止失败`

+   `重新启动失败`

+   `无法启动备份`

+   `无法中止备份`

+   `无法进入单用户模式`

+   `无法退出单用户模式`

+   `无法完成配置更改`

+   `获取配置失败`

+   `使用错误`

+   `成功`

+   `永久错误`

+   `临时错误`

+   `未知结果`

+   `临时错误，重新启动节点`

+   `永久错误，需要外部操作`

+   `Ndbd 文件系统错误，重新启动节点初始`

+   `未知`

error_classification 列显示错误分类。有关分类代码及其含义的信息，请参阅 NDB 错误分类。
