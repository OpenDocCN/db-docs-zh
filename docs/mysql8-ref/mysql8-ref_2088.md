> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-keyring-component-status-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-keyring-component-status-table.html)

#### 29.12.18.1 密钥环组件状态表

`keyring_component_status` 表（自 MySQL 8.0.24 起可用）提供有关正在使用的密钥环组件属性的状态信息，如果已安装密钥环组件。如果未安装密钥环组件（例如，如果未使用密钥环，或者配置为使用密钥环插件而不是密钥环组件管理密钥库），则表为空。

没有固定的属性集。每个密钥环组件可以自由定义自己的属性集。

示例 `keyring_component_status` 内容：

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

`keyring_component_status` 表具有以下列：

+   `STATUS_KEY`

    状态项名称。

+   `STATUS_VALUE`

    状态项值。

`keyring_component_status` 表没有索引。

`TRUNCATE TABLE` 不允许用于 `keyring_component_status` 表。
