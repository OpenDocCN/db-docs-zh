> 原文：[`dev.mysql.com/doc/refman/8.0/en/version-tokens-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-installation.html)

#### 7.6.6.2 安装或卸载版本标记

注意

如果安装了版本标记，会涉及一些开销。为避免这种开销，请不要安装它，除非您打算使用它。

本节描述了如何安装或卸载版本标记（Version Tokens），它是在一个包含插件和可加载函数的插件库文件中实现的。有关安装或卸载插件和可加载函数的一般信息，请参见第 7.6.1 节，“安装和卸载插件”，以及第 7.7.1 节，“安装和卸载可加载函数”。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来配置插件目录位置。

插件库文件基本名称为`version_tokens`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为`.so`，对于 Windows 为`.dll`）。

要安装版本标记插件和函数，请使用`INSTALL PLUGIN`和`CREATE FUNCTION`语句，根据需要调整平台的`.so`后缀：

```sql
INSTALL PLUGIN version_tokens SONAME 'version_token.so';
CREATE FUNCTION version_tokens_set RETURNS STRING
  SONAME 'version_token.so';
CREATE FUNCTION version_tokens_show RETURNS STRING
  SONAME 'version_token.so';
CREATE FUNCTION version_tokens_edit RETURNS STRING
  SONAME 'version_token.so';
CREATE FUNCTION version_tokens_delete RETURNS STRING
  SONAME 'version_token.so';
CREATE FUNCTION version_tokens_lock_shared RETURNS INT
  SONAME 'version_token.so';
CREATE FUNCTION version_tokens_lock_exclusive RETURNS INT
  SONAME 'version_token.so';
CREATE FUNCTION version_tokens_unlock RETURNS INT
  SONAME 'version_token.so';
```

您必须安装用于管理服务器版本标记列表的函数，但也必须安装插件，因为没有插件，函数无法正常工作。

如果插件和函数在复制源服务器上使用，请在所有副本服务器上安装它们，以避免复制问题。

一旦按照上述方式安装，插件和函数将保持安装状态直到卸载。要移除它们，请使用`UNINSTALL PLUGIN`和`DROP FUNCTION`语句：

```sql
UNINSTALL PLUGIN version_tokens;
DROP FUNCTION version_tokens_set;
DROP FUNCTION version_tokens_show;
DROP FUNCTION version_tokens_edit;
DROP FUNCTION version_tokens_delete;
DROP FUNCTION version_tokens_lock_shared;
DROP FUNCTION version_tokens_lock_exclusive;
DROP FUNCTION version_tokens_unlock;
```
