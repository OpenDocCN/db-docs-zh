> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-installation.html)

#### 8.4.5.2 安装或卸载 MySQL Enterprise Audit

本节描述了如何安装或卸载 MySQL Enterprise Audit，该功能使用审计日志插件和相关元素实现，详见第 8.4.5.1 节，“MySQL Enterprise Audit 的元素”。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

在升级 MySQL 安装时，插件升级不是自动的，一些插件可加载函数必须手动加载（参见安装可加载函数）。或者，您可以在升级 MySQL 后重新安装插件以加载新函数。

重要

在按照说明操作之前，请阅读本节的全部内容。根据您的环境，过程的某些部分可能有所不同。

注意

如果安装了`audit_log`插件，即使禁用了，也会带来一些最小的开销。为了避免这种开销，请不要安装 MySQL Enterprise Audit，除非你打算使用它。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如果需要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

要安装 MySQL Enterprise Audit，请查看您的 MySQL 安装的`share`目录，并选择适合您平台的脚本。可用的脚本在用于引用脚本的文件名上有所不同：

+   `audit_log_filter_win_install.sql`

+   `audit_log_filter_linux_install.sql`

按照以下方式运行脚本。这里的示例使用 Linux 安装脚本。请根据您的系统进行适当的替换。

在 MySQL 8.0.34 之前：

```sql
$> mysql -u root -p < audit_log_filter_linux_install.sql
Enter password: *(enter root password here)*
```

MySQL 8.0.34 及更高版本：

```sql
$> mysql -u root -p -D mysql < audit_log_filter_linux_install.sql
Enter password: *(enter root password here)*
```

从 MySQL 8.0.34 开始，可以在运行安装脚本时选择用于存储 JSON 过滤表的数据库。首先创建数据库；其名称不应超过 64 个字符。例如：

```sql
mysql> CREATE DATABASE IF NOT EXISTS *database-name*;
```

接下来，使用替代数据库名称运行脚本。

```sql
$> mysql -u root -p -D *database-name* < audit_log_filter_linux_install.sql
Enter password: *(enter root password here)*
```

注意

一些 MySQL 版本已经对 MySQL Enterprise Audit 表的结构进行了更改。为了确保您的表在从 MySQL 8.0 的早期版本升级时是最新的，请执行 MySQL 升级过程，确保使用强制更新选项（参见第三章，“升级 MySQL”）。如果您希望仅对 MySQL Enterprise Audit 表运行更新语句，请参阅以下讨论。

截至 MySQL 8.0.12 版本，对于新的 MySQL 安装，MySQL 企业审计使用的`audit_log_user`表中的`USER`和`HOST`列的定义更好地对应于`mysql.user`系统表中`User`和`Host`列的定义。对于已安装 MySQL 企业审计的升级安装，建议您按照以下方式修改表定义：

```sql
ALTER TABLE mysql.audit_log_user
  DROP FOREIGN KEY audit_log_user_ibfk_1;
ALTER TABLE mysql.audit_log_filter
  CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_as_ci;
ALTER TABLE mysql.audit_log_user
  CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_as_ci;
ALTER TABLE mysql.audit_log_user
  MODIFY COLUMN USER VARCHAR(32);
ALTER TABLE mysql.audit_log_user
  ADD FOREIGN KEY (FILTERNAME) REFERENCES mysql.audit_log_filter(NAME);
```

注意

要在源/副本复制、组复制或 InnoDB 集群的上下文中使用 MySQL 企业审计，必须在源节点上运行安装脚本之前准备好副本节点。这是必要的，因为脚本中的`INSTALL PLUGIN`语句不会被复制。

1.  在每个副本节点上，从安装脚本中提取`INSTALL PLUGIN`语句，并手动执行它。

1.  在源节点上，按照先前描述的方式运行安装脚本。

要验证插件安装，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见 Section 7.6.2, “Obtaining Server Plugin Information”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'audit%';
+-------------+---------------+
| PLUGIN_NAME | PLUGIN_STATUS |
+-------------+---------------+
| audit_log   | ACTIVE        |
+-------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断消息。

安装了 MySQL 企业审计后，您可以在后续服务器启动时使用`--audit-log`选项来控制`audit_log`插件的激活。例如，为了防止插件在运行时被移除，请使用此选项：

```sql
[mysqld]
audit-log=FORCE_PLUS_PERMANENT
```

如果希望防止服务器在没有审计插件的情况下运行，请使用`--audit-log`并将值设置为`FORCE`或`FORCE_PLUS_PERMANENT`，以强制服务器启动失败，如果插件初始化失败。

重要

默认情况下，基于规则的审计日志过滤不会记录任何用户的可审计事件。这与传统审计日志行为不同，传统审计日志会记录所有用户的所有可审计事件（参见 Section 8.4.5.10, “Legacy Mode Audit Log Filtering”）。如果希望通过基于规则的过滤产生记录所有内容的行为，请创建一个简单的过滤器以启用日志记录，并将其分配给默认帐户：

```sql
SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');
SELECT audit_log_filter_set_user('%', 'log_all');
```

分配给`%`的过滤器用于来自没有明确分配过滤器的任何帐户的连接（最初对所有帐户都是真实的）。

当按照上述描述安装后，MySQL 企业审计将一直保留安装状态，直到被卸载。要在 MySQL 8.0.35 及更高版本中移除它，请运行位于 MySQL 安装的`share`目录中的卸载脚本。这里的示例指定了默认系统数据库`mysql`，请根据您的系统进行适当的替换。

```sql
$> mysql -u root -p -D mysql < audit_log_filter_uninstall.sql
Enter password: *(enter root password here)*
```

如果脚本不可用，请执行以下语句手动移除表、插件和函数。

```sql
DROP TABLE IF EXISTS mysql.audit_log_user;
DROP TABLE IF EXISTS mysql.audit_log_filter;
UNINSTALL PLUGIN audit_log;
DROP FUNCTION audit_log_filter_set_filter;
DROP FUNCTION audit_log_filter_remove_filter;
DROP FUNCTION audit_log_filter_set_user;
DROP FUNCTION audit_log_filter_remove_user;
DROP FUNCTION audit_log_filter_flush;
DROP FUNCTION audit_log_encryption_password_get;
DROP FUNCTION audit_log_encryption_password_set;
DROP FUNCTION audit_log_read;
DROP FUNCTION audit_log_read_bookmark;
DROP FUNCTION audit_log_rotate;
```
