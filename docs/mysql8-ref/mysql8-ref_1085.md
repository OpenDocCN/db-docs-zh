> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-procedure.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-procedure.html)

#### 15.7.7.9 显示创建存储过程语句

```sql
SHOW CREATE PROCEDURE *proc_name*
```

此语句是 MySQL 的扩展。它返回一个确切的字符串，可用于重新创建指定的存储过程。类似的语句，`显示创建函数`，显示有关存储函数的信息（参见第 15.7.7.8 节，“显示创建函数语句”）。

要使用任一语句，您必须是例程`DEFINER`的命名用户，具有`SHOW_ROUTINE`权限，在全局级别具有`SELECT`权限，或者在包括例程的范围内被授予`CREATE ROUTINE`、`ALTER ROUTINE`或`EXECUTE`权限。如果您只具有`CREATE ROUTINE`、`ALTER ROUTINE`或`EXECUTE`权限，则`Create Procedure`或`Create Function`字段显示的值为`NULL`。

```sql
mysql> SHOW CREATE PROCEDURE test.citycount\G
*************************** 1\. row ***************************
           Procedure: citycount
            sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                      NO_ZERO_IN_DATE,NO_ZERO_DATE,
                      ERROR_FOR_DIVISION_BY_ZERO,
                      NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`me`@`localhost`
                      PROCEDURE `citycount`(IN country CHAR(3), OUT cities INT)
                      BEGIN
                        SELECT COUNT(*) INTO cities FROM world.city
                        WHERE CountryCode = country;
                      END
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci 
mysql> SHOW CREATE FUNCTION test.hello\G
*************************** 1\. row ***************************
            Function: hello
            sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,
                      NO_ZERO_IN_DATE,NO_ZERO_DATE,
                      ERROR_FOR_DIVISION_BY_ZERO,
                      NO_ENGINE_SUBSTITUTION
     Create Function: CREATE DEFINER=`me`@`localhost`
                      FUNCTION `hello`(s CHAR(20))
                      RETURNS char(50) CHARSET utf8mb4
                      DETERMINISTIC
                      RETURN CONCAT('Hello, ',s,'!')
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
```

`character_set_client`是创建例程时的`character_set_client`系统变量的会话值。`collation_connection`是创建例程时的`collation_connection`系统变量的会话值。`数据库排序规则`是与例程关联的数据库的排序规则。
