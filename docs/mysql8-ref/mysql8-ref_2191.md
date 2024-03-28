> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-create-synonym-db.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-create-synonym-db.html)

#### 30.4.4.1 create_synonym_db() 过程

给定一个模式名称，此过程将创建一个包含引用原始模式中所有表和视图的视图的同义词模式。例如，可以使用此方法创建一个更短的名称来引用具有长名称的模式（例如使用`info`而不是`INFORMATION_SCHEMA`）。

##### 参数

+   `in_db_name VARCHAR(64)`: 要创建同义词的模式的名称。

+   `in_synonym VARCHAR(64)`: 用于同义词模式的名称。此模式不能已经存在。

##### 示例

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| world              |
+--------------------+
mysql> CALL sys.create_synonym_db('INFORMATION_SCHEMA', 'info');
+---------------------------------------+
| summary                               |
+---------------------------------------+
| Created 63 views in the info database |
+---------------------------------------+
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| info               |
| mysql              |
| performance_schema |
| sys                |
| world              |
+--------------------+
mysql> SHOW FULL TABLES FROM info;
+---------------------------------------+------------+
| Tables_in_info                        | Table_type |
+---------------------------------------+------------+
| character_sets                        | VIEW       |
| collation_character_set_applicability | VIEW       |
| collations                            | VIEW       |
| column_privileges                     | VIEW       |
| columns                               | VIEW       |
...
```
