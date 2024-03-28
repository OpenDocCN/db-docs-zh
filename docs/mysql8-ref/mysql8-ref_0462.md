> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-plugin-installation.html)

#### 8.5.3.1 MySQL 企业数据脱敏和去标识化插件安装

本节描述了如何安装或卸载 MySQL 企业数据脱敏和去标识化，它是作为一个包含插件和几个可加载函数的插件库文件实现的。有关安装或卸载插件和可加载函数的一般信息，请参见 Section 7.6.1, “Installing and Uninstalling Plugins”和 Section 7.7.1, “Installing and Uninstalling Loadable Functions”。

要被服务器使用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如有必要，在服务器启动时通过设置`plugin_dir`的值来配置插件目录位置。

插件库文件的基本名称是`data_masking`。文件名后缀因平台而异（例如，Unix 和类 Unix 系统使用`.so`，Windows 使用`.dll`）。

要安装 MySQL 企业数据脱敏和去标识化插件和函数，请使用`INSTALL PLUGIN`和`CREATE FUNCTION`语句，并根据需要调整平台的`.so`后缀：

```sql
INSTALL PLUGIN data_masking SONAME 'data_masking.so';
CREATE FUNCTION gen_blocklist RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_dictionary RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_dictionary_drop RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_dictionary_load RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_range RETURNS INTEGER
  SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_email RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_pan RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_ssn RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION gen_rnd_us_phone RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION mask_inner RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION mask_outer RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION mask_pan RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION mask_pan_relaxed RETURNS STRING
  SONAME 'data_masking.so';
CREATE FUNCTION mask_ssn RETURNS STRING
  SONAME 'data_masking.so';
```

如果插件和函数在复制源服务器上使用，则还需在所有副本服务器上安装它们，以避免复制问题。

一旦按照上述描述安装，插件和函数将保持安装状态，直到被卸载。要移除它们，请使用`UNINSTALL PLUGIN`和`DROP FUNCTION`语句：

```sql
UNINSTALL PLUGIN data_masking;
DROP FUNCTION gen_blocklist;
DROP FUNCTION gen_dictionary;
DROP FUNCTION gen_dictionary_drop;
DROP FUNCTION gen_dictionary_load;
DROP FUNCTION gen_range;
DROP FUNCTION gen_rnd_email;
DROP FUNCTION gen_rnd_pan;
DROP FUNCTION gen_rnd_ssn;
DROP FUNCTION gen_rnd_us_phone;
DROP FUNCTION mask_inner;
DROP FUNCTION mask_outer;
DROP FUNCTION mask_pan;
DROP FUNCTION mask_pan_relaxed;
DROP FUNCTION mask_ssn;
```
