> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-database.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-database.html)

#### 15.7.7.6 显示创建数据库语句

```sql
SHOW CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] *db_name*
```

显示创建指定数据库的`CREATE DATABASE`语句。如果`SHOW`语句包含`IF NOT EXISTS`子句，则输出也包含此子句。`SHOW CREATE SCHEMA`是`SHOW CREATE DATABASE`的同义词。

```sql
mysql> SHOW CREATE DATABASE test\G
*************************** 1\. row ***************************
       Database: test
Create Database: CREATE DATABASE `test` /*!40100 DEFAULT CHARACTER SET utf8mb4
                 COLLATE utf8mb4_0900_ai_ci */ /*!80014 DEFAULT ENCRYPTION='N' */ 
mysql> SHOW CREATE SCHEMA test\G
*************************** 1\. row ***************************
       Database: test
Create Database: CREATE DATABASE `test` /*!40100 DEFAULT CHARACTER SET utf8mb4
                 COLLATE utf8mb4_0900_ai_ci */ /*!80014 DEFAULT ENCRYPTION='N' */
```

`SHOW CREATE DATABASE`根据`sql_quote_show_create`选项的值引用表名和列名。参见第 7.1.8 节，“服务器系统变量”。
