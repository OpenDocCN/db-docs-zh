> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-metadata.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-metadata.html)

#### 8.4.4.17 密钥环元数据

本节描述了关于密钥环使用的信息来源。

要查看密钥环插件是否已加载，请检查信息模式`PLUGINS`表或使用`SHOW PLUGINS`语句（参见第 7.6.2 节，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'keyring%';
+--------------+---------------+
| PLUGIN_NAME  | PLUGIN_STATUS |
+--------------+---------------+
| keyring_file | ACTIVE        |
+--------------+---------------+
```

要查看存在哪些密钥，请检查性能模式`keyring_keys`表：

```sql
mysql> SELECT * FROM performance_schema.keyring_keys;
+-----------------------------+--------------+----------------+
| KEY_ID                      | KEY_OWNER    | BACKEND_KEY_ID |
+-----------------------------+--------------+----------------+
| audit_log-20210322T130749-1 |              |                |
| MyKey                       | me@localhost |                |
| YourKey                     | me@localhost |                |
+-----------------------------+--------------+----------------+
```

要查看密钥环组件是否已加载，请检查性能模式`keyring_component_status`表。例如：

```sql
mysql> SELECT * FROM performance_schema.keyring_component_status;
+---------------------+-------------------------------------------------+
| STATUS_KEY          | STATUS_VALUE                                    |
+---------------------+-------------------------------------------------+
| Component_name      | component_keyring_file                          |
| Author              | Oracle Corporation                              |
| License             | GPL                                             |
| Implementation_name | component_keyring_file                          |
| Version             | 1.0                                             |
| Component_status    | Active                                          |
| Data_file           | /usr/local/mysql/keyring/component_keyring_file |
| Read_only           | No                                              |
+---------------------+-------------------------------------------------+
```

`Component_status`值为`Active`表示组件初始化成功。如果组件加载但初始化失败，则值为`Disabled`。
