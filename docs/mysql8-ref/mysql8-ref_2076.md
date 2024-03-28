> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-persisted-variables-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-persisted-variables-table.html)

#### 29.12.14.1 性能模式 persisted_variables 表

`persisted_variables`表提供了一个 SQL 接口，用于存储持久化全局系统变量设置的`mysqld-auto.cnf`文件，使得可以使用`SELECT`语句在运行时检查文件内容。变量使用`SET PERSIST`或`PERSIST_ONLY`语句进行持久化；请参阅第 15.7.6.1 节，“变量赋值的 SET 语法”。该表中包含文件中每个持久化系统变量的一行。未持久化的变量不会出现在表中。

查看此表中敏感系统变量的值需要`SENSITIVE_VARIABLES_OBSERVER`权限。

关于持久化系统变量的信息，请参阅第 7.1.9.3 节，“持久化系统变量”。

假设`mysqld-auto.cnf`看起来像这样（稍作重新格式化）：

```sql
{
  "Version": 1,
  "mysql_server": {
    "max_connections": {
      "Value": "1000",
      "Metadata": {
        "Timestamp": 1.519921706e+15,
        "User": "root",
        "Host": "localhost"
      }
    },
    "autocommit": {
      "Value": "ON",
      "Metadata": {
        "Timestamp": 1.519921707e+15,
        "User": "root",
        "Host": "localhost"
      }
    }
  }
}
```

然后`persisted_variables`具有以下内容：

```sql
mysql> SELECT * FROM performance_schema.persisted_variables;
+-----------------+----------------+
| VARIABLE_NAME   | VARIABLE_VALUE |
+-----------------+----------------+
| autocommit      | ON             |
| max_connections | 1000           |
+-----------------+----------------+
```

`persisted_variables`表具有以下列：

+   `变量名`

    在`mysqld-auto.cnf`中列出的变量名。

+   `变量值`

    在`mysqld-auto.cnf`中列出的变量值。

`persisted_variables`具有以下索引：

+   在(`变量名`)上的主键

`TRUNCATE TABLE`不允许用于`persisted_variables`表。
