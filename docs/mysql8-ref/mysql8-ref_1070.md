> 原文：[`dev.mysql.com/doc/refman/8.0/en/uninstall-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/uninstall-plugin.html)

#### 15.7.4.6 UNINSTALL PLUGIN Statement

```sql
UNINSTALL PLUGIN *plugin_name*
```

此语句移除已安装的服务器插件。`UNINSTALL PLUGIN`是`INSTALL PLUGIN`的补充。它需要对`mysql.plugin`系统表的`DELETE`权限，因为它会从该表中删除注册插件的行。

*`plugin_name`*必须是`mysql.plugin`表中列出的某个插件的名称。服务器执行插件的去初始化函数，并从`mysql.plugin`系统表中删除插件的行，以便后续服务器重新启动时不加载和初始化插件。`UNINSTALL PLUGIN`不会删除插件的共享库文件。

如果使用插件的任何表是打开状态，则无法卸载插件。

插件的移除对关联表的使用有影响。例如，如果一个全文解析器插件与表上的`FULLTEXT`索引相关联，卸载插件会使表无法使用。任何尝试访问该表的操作都会导致错误。甚至无法打开表，因此无法删除使用该插件的索引。这意味着慎重卸载插件，除非你不在乎表的内容。如果你打算卸载插件而不打算以后重新安装它，并且你关心表的内容，你应该使用**mysqldump**导出表，并从导出的`CREATE TABLE`语句中删除`WITH PARSER`子句，以便以后重新加载表。如果你不在乎表，即使表上关联的插件丢失，也可以使用`DROP TABLE`。

有关插件加载的更多信息，请参见 Section 7.6.1, “Installing and Uninstalling Plugins”。
