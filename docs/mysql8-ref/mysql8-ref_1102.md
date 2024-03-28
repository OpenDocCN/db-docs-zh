> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-privileges.html`](https://dev.mysql.com/doc/refman/8.0/en/show-privileges.html)

#### 15.7.7.26 SHOW PRIVILEGES Statement

```sql
SHOW PRIVILEGES
```

`SHOW PRIVILEGES` 显示了 MySQL 服务器支持的系统权限列表。显示的权限包括所有静态权限和当前注册的动态权限。

```sql
mysql> SHOW PRIVILEGES\G
*************************** 1\. row ***************************
Privilege: Alter
  Context: Tables
  Comment: To alter the table
*************************** 2\. row ***************************
Privilege: Alter routine
  Context: Functions,Procedures
  Comment: To alter or drop stored functions/procedures
*************************** 3\. row ***************************
Privilege: Create
  Context: Databases,Tables,Indexes
  Comment: To create new databases and tables
*************************** 4\. row ***************************
Privilege: Create routine
  Context: Databases
  Comment: To use CREATE FUNCTION/PROCEDURE
*************************** 5\. row ***************************
Privilege: Create temporary tables
  Context: Databases
  Comment: To use CREATE TEMPORARY TABLE
...
```

特定用户拥有的权限可以通过 `SHOW GRANTS` 语句显示。更多信息请参见 Section 15.7.7.21, “SHOW GRANTS Statement”。
