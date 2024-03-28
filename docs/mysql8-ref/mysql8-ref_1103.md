> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-procedure-code.html`](https://dev.mysql.com/doc/refman/8.0/en/show-procedure-code.html)

#### 15.7.7.27 显示存储过程代码语句

```sql
SHOW PROCEDURE CODE *proc_name*
```

此语句是 MySQL 的扩展，仅适用于已使用调试支持构建的服务器。它显示了命名存储过程的内部实现的表示。类似的语句，`SHOW FUNCTION CODE`，显示有关存储函数的信息（请参阅第 15.7.7.19 节，“显示函数代码语句”）。

要使用任一语句，您必须是以例程`DEFINER`命名的用户，具有`SHOW_ROUTINE`权限，或者在全局级别具有`SELECT`权限。

如果命名例程可用，则每个语句都会生成一个结果集。结果集中的每一行对应于例程中的一个“指令”。第一列是`Pos`，它是从 0 开始的序号。第二列是`Instruction`，其中包含一个 SQL 语句（通常是从原始源更改而来），或者仅对存储过程处理程序有意义的指令。

```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE p1 ()
       BEGIN
         DECLARE fanta INT DEFAULT 55;
         DROP TABLE t2;
         LOOP
           INSERT INTO t3 VALUES (fanta);
           END LOOP;
         END//
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW PROCEDURE CODE p1//
+-----+----------------------------------------+
| Pos | Instruction                            |
+-----+----------------------------------------+
|   0 | set fanta@0 55                         |
|   1 | stmt 9 "DROP TABLE t2"                 |
|   2 | stmt 5 "INSERT INTO t3 VALUES (fanta)" |
|   3 | jump 2                                 |
+-----+----------------------------------------+
4 rows in set (0.00 sec)

mysql> CREATE FUNCTION test.hello (s CHAR(20))
       RETURNS CHAR(50) DETERMINISTIC
       RETURN CONCAT('Hello, ',s,'!');
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW FUNCTION CODE test.hello;
+-----+---------------------------------------+
| Pos | Instruction                           |
+-----+---------------------------------------+
|   0 | freturn 254 concat('Hello, ',s@0,'!') |
+-----+---------------------------------------+
1 row in set (0.00 sec)
```

在此示例中，不可执行的`BEGIN`和`END`语句已消失，对于`DECLARE *`variable_name`*`语句，仅显示可执行部分（分配默认值的部分）。对于从源中提取的每个语句，都有一个代码词`stmt`，后跟一个类型（9 表示`DROP`，5 表示`INSERT`，依此类推）。最后一行包含指令`jump 2`，表示`GOTO 指令＃2`。
