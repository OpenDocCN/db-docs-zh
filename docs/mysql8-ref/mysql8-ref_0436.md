> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-elements.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-elements.html)

#### 8.4.5.1 MySQL 企业审计的元素

MySQL 企业审计基于审计日志插件和相关元素：

+   一个名为`audit_log`的服务器端插件检查可审计事件，并确定是否将其写入审计日志。

+   一组函数使得可以操作控制日志行为的过滤定义、加密密码和日志文件读取。

+   `mysql`系统数据库中的表提供过滤和用户账户数据的持久存储，除非您在服务器启动时设置`audit_log_database`系统变量以指定不同的数据库。

+   系统变量使审计日志配置生效，状态变量提供运行时操作信息。

+   `AUDIT_ADMIN`权限使用户能够管理审计日志，并且（从 MySQL 8.0.28 开始）`AUDIT_ABORT_EXEMPT`权限使系统用户能够执行否则会被审计日志过滤器中的“中止”项阻止的查询。
