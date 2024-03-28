# 7.5.2 获取组件信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/obtaining-component-information.html`](https://dev.mysql.com/doc/refman/8.0/en/obtaining-component-information.html)

`mysql.component` 系统表包含有关当前加载的组件的信息，并显示哪些组件已使用`INSTALL COMPONENT`进行注册。从表中选择显示已安装的组件。例如：

```sql
mysql> SELECT * FROM mysql.component;
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password |
|            2 |                  2 | file://component_log_sink_json     |
+--------------+--------------------+------------------------------------+
```

`component_id` 和 `component_group_id` 值仅供内部使用。`component_urn` 是在`INSTALL COMPONENT`和`UNINSTALL COMPONENT`语句中用于加载和卸载组件的 URN。
