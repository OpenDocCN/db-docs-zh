# 18.11.1 可插拔存储引擎架构

> 原文：[`dev.mysql.com/doc/refman/8.0/en/pluggable-storage.html`](https://dev.mysql.com/doc/refman/8.0/en/pluggable-storage.html)

MySQL 服务器使用可插拔存储引擎架构，允许将存储引擎加载到运行中的 MySQL 服务器中并从中卸载。

**安装存储引擎**

在使用存储引擎之前，必须使用`INSTALL PLUGIN`语句将存储引擎插件共享库加载到 MySQL 中。例如，如果`EXAMPLE`引擎插件命名为`example`，共享库命名为`ha_example.so`，则可以使用以下语句加载：

```sql
INSTALL PLUGIN example SONAME 'ha_example.so';
```

要安装可插拔存储引擎，插件文件必须位于 MySQL 插件目录中，并且发出`INSTALL PLUGIN`语句的用户必须对`mysql.plugin`表具有`INSERT`权限。

共享库必须位于 MySQL 服务器插件目录中，其位置由`plugin_dir`系统变量给出。

**卸载存储引擎**

要卸载存储引擎，请使用`UNINSTALL PLUGIN`语句：

```sql
UNINSTALL PLUGIN example;
```

如果卸载一个现有表需要的存储引擎，那些表将变得无法访问，但仍然存在于磁盘上（如果适用）。在卸载存储引擎之前，请确保没有表使用存储引擎。
