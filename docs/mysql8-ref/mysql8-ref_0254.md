# 7.1.17 服务器端帮助支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/server-side-help-support.html`](https://dev.mysql.com/doc/refman/8.0/en/server-side-help-support.html)

MySQL 服务器支持一个`HELP`语句，该语句从 MySQL 参考手册中返回信息（参见 Section 15.8.3, “HELP Statement”）。这些信息存储在`mysql`模式的几个表中（参见 Section 7.3, “The mysql System Schema”）。`HELP`语句的正确操作要求这些帮助表被初始化。

对于在 Unix 上使用二进制或源发行版进行 MySQL 的新安装，帮助表内容初始化发生在初始化数据目录时（参见 Section 2.9.1, “Initializing the Data Directory”）。对于 Linux 上的 RPM 发行版或 Windows 上的二进制发行版，内容初始化作为 MySQL 安装过程的一部分发生。

对于使用二进制发行版进行 MySQL 升级，截至 MySQL 8.0.16，帮助表内容会被服务器自动升级。在 MySQL 8.0.16 之前，内容不会自动升级，但你可以手动升级。在`share`或`share/mysql`目录中找到`fill_help_tables.sql`文件。进入该目录并使用**mysql**客户端处理文件，如下所示：

```sql
mysql -u root -p mysql < fill_help_tables.sql
```

此处显示的命令假定你使用具有修改`mysql`模式中表权限的帐户（如`root`）连接到服务器。根据需要调整连接参数。

在 MySQL 8.0.16 之前，如果你正在使用 Git 和 MySQL 开发源码树，源码树只包含一个“存根”版本的`fill_help_tables.sql`。要获取非存根版本，请使用源或二进制发行版中的一个。

注意

每个 MySQL 系列都有自己系列特定的参考手册，因此帮助表内容也是系列特定的。这对于复制有影响，因为帮助表内容应该与 MySQL 系列匹配。如果你将 MySQL 8.0 帮助内容加载到 MySQL 8.0 复制服务器中，将这些内容复制到来自不同 MySQL 系列且不适用于该内容的副本服务器是没有意义的。因此，当你在复制场景中升级单个服务器时，应该根据前面给出的说明升级每个服务器的帮助表。（只有低于 8.0.16 版本的复制服务器才需要手动升级帮助内容。如前述说明所述，从 MySQL 8.0.16 开始，内容升级会自动发生。）
