# 28.3.22 `INFORMATION_SCHEMA PLUGINS` 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-plugins-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-plugins-table.html)

`PLUGINS` 表提供有关服务器插件的信息。

`PLUGINS` 表具有以下列：

+   `PLUGIN_NAME`

    在诸如 `INSTALL PLUGIN` 和 `UNINSTALL PLUGIN` 等语句中用于引用插件的名称。

+   `PLUGIN_VERSION`

    插件的一般类型描述符中的版本。

+   `PLUGIN_STATUS`

    插件状态，为 `ACTIVE`、`INACTIVE`、`DISABLED`、`DELETING` 或 `DELETED`。

+   `PLUGIN_TYPE`

    插件类型，如 `STORAGE ENGINE`、`INFORMATION_SCHEMA` 或 `AUTHENTICATION`。

+   `PLUGIN_TYPE_VERSION`

    插件的特定类型描述符中的版本。

+   `PLUGIN_LIBRARY`

    插件共享库文件的名称。这是在诸如 `INSTALL PLUGIN` 和 `UNINSTALL PLUGIN` 等语句中用于引用插件文件的名称。此文件位于由 `plugin_dir` 系统变量命名的目录中。如果库名称为 `NULL`，则插件已编译并且无法使用 `UNINSTALL PLUGIN` 卸载。

+   `PLUGIN_LIBRARY_VERSION`

    插件 API 接口版本。

+   `PLUGIN_AUTHOR`

    插件作者。

+   `PLUGIN_DESCRIPTION`

    插件的简要描述。

+   `PLUGIN_LICENSE`

    插件许可证（例如，`GPL`）。

+   `LOAD_OPTION`

    插件加载方式。值为 `OFF`、`ON`、`FORCE` 或 `FORCE_PLUS_PERMANENT`。参见 Section 7.6.1, “Installing and Uninstalling Plugins”。

#### 注意事项

+   `PLUGINS` 是一个非标准的 `INFORMATION_SCHEMA` 表。

+   对于使用 `INSTALL PLUGIN` 安装的插件，`PLUGIN_NAME` 和 `PLUGIN_LIBRARY` 值也会在 `mysql.plugin` 表中注册。

+   有关构��� `PLUGINS` 表信息基础的插件数据结构，请参阅 MySQL 插件 API。

插件信息也可以通过 `SHOW PLUGINS` 语句获取。参见 Section 15.7.7.25, “SHOW PLUGINS Statement”。这些语句是等效的：

```sql
SELECT
  PLUGIN_NAME, PLUGIN_STATUS, PLUGIN_TYPE,
  PLUGIN_LIBRARY, PLUGIN_LICENSE
FROM INFORMATION_SCHEMA.PLUGINS;

SHOW PLUGINS;
```
