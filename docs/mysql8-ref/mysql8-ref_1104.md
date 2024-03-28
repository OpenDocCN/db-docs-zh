> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-procedure-status.html`](https://dev.mysql.com/doc/refman/8.0/en/show-procedure-status.html)

#### 15.7.7.28 显示存储过程状态语句

```sql
SHOW PROCEDURE STATUS
    [LIKE '*pattern*' | WHERE *expr*]
```

此语句是 MySQL 的扩展。它返回存储过程的特征，如数据库、名称、类型、创建者、创建和修改日期以及字符集信息。类似的语句，`SHOW FUNCTION STATUS`，显示有关存储函数的信息（参见第 15.7.7.20 节，“显示函数状态语句”）。

要使用任一语句，您必须是以例行`DEFINER`身份命名的用户，具有`SHOW_ROUTINE`权限，在全局级别具有`SELECT`权限，或者在包含例程的范围内被授予`CREATE ROUTINE`，`ALTER ROUTINE`，或`EXECUTE`权限。

如果存在`LIKE`子句，则指示要匹配的过程或函数名称。可以使用`WHERE`子句选择使用更一般条件的行，如第 28.8 节，“SHOW 语句的扩展”中所讨论的。

```sql
mysql> SHOW PROCEDURE STATUS LIKE 'sp1'\G
*************************** 1\. row ***************************
                  Db: test
                Name: sp1
                Type: PROCEDURE
             Definer: testuser@localhost
            Modified: 2018-08-08 13:54:11
             Created: 2018-08-08 13:54:11
       Security_type: DEFINER
             Comment:
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci 
mysql> SHOW FUNCTION STATUS LIKE 'hello'\G
*************************** 1\. row ***************************
                  Db: test
                Name: hello
                Type: FUNCTION
             Definer: testuser@localhost
            Modified: 2020-03-10 11:10:03
             Created: 2020-03-10 11:10:03
       Security_type: DEFINER
             Comment:
character_set_client: utf8mb4
collation_connection: utf8mb4_0900_ai_ci
  Database Collation: utf8mb4_0900_ai_ci
```

`character_set_client`是创建例程时的`character_set_client`系统变量的会话值。`collation_connection`是创建例程时的`collation_connection`系统变量的会话值。`数据库排序规则`是例程所关联的数据库的排序规则。

存储例程信息也可以从`INFORMATION_SCHEMA`的`PARAMETERS`和`ROUTINES`表中获取。请参见第 28.3.20 节，“INFORMATION_SCHEMA PARAMETERS 表”，以及第 28.3.30 节，“INFORMATION_SCHEMA ROUTINES 表”。
