> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-gr-memory-monitoring-ps-instruments-enable.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-gr-memory-monitoring-ps-instruments-enable.html)

#### 20.7.9.1 启用或禁用组复制仪器

要从命令行启用所有组复制仪器，请在您选择的 SQL 客户端中运行以下命令：

```sql
 UPDATE performance_schema.setup_instruments SET ENABLED = 'YES' 
        WHERE NAME LIKE 'memory/group_rpl/%';
```

要从命令行禁用所有组复制仪器，请在您选择的 SQL 客户端中运行以下命令：

```sql
 UPDATE performance_schema.setup_instruments SET ENABLED = 'NO' 
        WHERE NAME LIKE 'memory/group_rpl/%';
```

要在服务器启动时启用所有组复制仪器，请将以下内容添加到您的选项文件中：

```sql
 [mysqld]
 performance-schema-instrument='memory/group_rpl/%=ON'
```

要在服务器启动时禁用所有组复制仪器，请将以下内容添加到您的选项文件中：

```sql
 [mysqld]
 performance-schema-instrument='memory/group_rpl/%=OFF'
```

要启用或禁用该组中的单个仪器，请用该仪器的全名替换通配符（%）。

欲了解更多信息，请参阅 Section 29.3, “性能模式启动配置”和 Section 29.4, “性能模式运行时配置”。
