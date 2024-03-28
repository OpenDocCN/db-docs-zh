# 15.3.7 SET TRANSACTION 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-transaction.html`](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)

```sql
SET [GLOBAL | SESSION] TRANSACTION
    *transaction_characteristic* [, *transaction_characteristic*] ...

*transaction_characteristic*: {
    ISOLATION LEVEL *level*
  | *access_mode*
}

*level*: {
     REPEATABLE READ
   | READ COMMITTED
   | READ UNCOMMITTED
   | SERIALIZABLE
}

*access_mode*: {
     READ WRITE
   | READ ONLY
}
```

此语句指定了事务 特性。它接受一个由逗号分隔的一个或多个特性值的列表。每个特性值设置事务的隔离级别 或访问模式。隔离级别用于对`InnoDB` 表进行操作。访问模式指定事务是以读/写模式还是只读模式运行。

此外，`SET TRANSACTION` 可以包括一个可选的`GLOBAL`或`SESSION`关键字，以指示语句的范围。

+   事务隔离级别

+   事务访问模式

+   事务特性范围

#### 事务隔离级别

要设置事务隔离级别，请使用`ISOLATION LEVEL *`level`*`子句。不允许在同一`SET TRANSACTION` 语句中指定多个`ISOLATION LEVEL`子句。

默认隔离级别是`REPEATABLE READ`。其他允许的值是`READ COMMITTED`、`READ UNCOMMITTED` 和`SERIALIZABLE`。有关这些隔离级别的信息，请参见第 17.7.2.1 节，“事务隔离级别”。

#### 事务访问模式

要设置事务访问模式，请使用`READ WRITE`或`READ ONLY`子句。不允许在同一`SET TRANSACTION` 语句中指定多个访问模式子句。

默认情况下，事务以读/写模式进行，允许对事务中使用的表进行读取和写入。可以使用具有`READ WRITE`访问模式的`SET TRANSACTION` 明确指定此模式。

如果事务访问模式设置为`READ ONLY`，则禁止对表进行更改。这可能使存储引擎能够进行性能改进，这些改进在不允许写入时可能是可能的。

在只读模式下，仍然可以使用 DML 语句更改使用 `TEMPORARY` 关键字创建的表。使用 DDL 语句进行的更改是不允许的，就像永久表一样。

也可以使用 `START TRANSACTION` 语句为单个事务指定 `READ WRITE` 和 `READ ONLY` 访问模式。

#### 事务特性范围

您可以全局设置事务特性，为当前会话或仅限于下一个事务：

+   使用 `GLOBAL` 关键字：

    +   该语句全局适用于所有后续会话。

    +   现有会话不受影响。

+   使用 `SESSION` 关键字：

    +   该语句适用于当前会话中执行的所有后续事务。

    +   该语句允许在事务内部执行，但不影响当前正在进行的事务。

    +   如果在事务之间执行，则该语句将覆盖设置下一个事务值的任何先前语句。

+   没有任何 `SESSION` 或 `GLOBAL` 关键字：

    +   该语句仅适用于会话中执行的下一个单个事务。

    +   后续事务将恢复使用命名特性的会话值。

    +   该语句不允许在事务内部执行：

        ```sql
        mysql> START TRANSACTION;
        Query OK, 0 rows affected (0.02 sec)

        mysql> SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
        ERROR 1568 (25001): Transaction characteristics can't be changed
        while a transaction is in progress
        ```

更改全局事务特性需要 `CONNECTION_ADMIN` 权限（或已弃用的 `SUPER` 权限）。任何会话都可以自由更改其会话特性（即使在事务中间），或者更改其下一个事务的特性（在该事务开始之前）。

要在服务器启动时设置全局隔离级别，请在命令行或选项文件中使用 `--transaction-isolation=*`level`*` 选项。此选项的 *`level`* 值使用破折号而不是空格，因此可接受的值为 `READ-UNCOMMITTED`、`READ-COMMITTED`、`REPEATABLE-READ` 或 `SERIALIZABLE`。

同样，要在服务器启动时设置全局事务访问模式，请使用 `--transaction-read-only` 选项。默认值为 `OFF`（读/写模式），但该值可以设置为 `ON` 以进行只读模式。

例如，要将隔离级别设置为 `REPEATABLE READ` 并将访问模式设置为 `READ WRITE`，请在选项文件的 `[mysqld]` 部分中使用以下行：

```sql
[mysqld]
transaction-isolation = REPEATABLE-READ
transaction-read-only = OFF
```

在运行时，可以间接地使用`SET TRANSACTION`语句设置全局、会话和下一个事务范围级别的特征，如前所述。也可以直接使用`SET`语句为`transaction_isolation`和`transaction_read_only`系统变量赋值来直接设置它们：

+   `SET TRANSACTION`允许在不同范围级别设置事务特征时使用可选的`GLOBAL`和`SESSION`关键字。

+   用于为`transaction_isolation`和`transaction_read_only`系统变量赋值的`SET`语句具有在不同范围级别设置这些变量的语法。

以下表格显示了每个`SET TRANSACTION`和变量赋值语法设置的特征范围级别。

**表 15.9 事务特征的 SET TRANSACTION 语法**

| 语法 | 受影响的特征范围 |
| --- | --- |
| `SET GLOBAL TRANSACTION *`transaction_characteristic`*` | 全局 |
| `SET SESSION TRANSACTION *`transaction_characteristic`*` | 会话 |
| `SET TRANSACTION *`transaction_characteristic`*` | 仅适用于下一个事务 |

**表 15.10 事务特征的 SET 语法**

| 语法 | 受影响的特征范围 |
| --- | --- |
| `SET GLOBAL *`var_name`* = *`value`*` | 全局 |
| `SET @@GLOBAL.*`var_name`* = *`value`*` | 全局 |
| `SET PERSIST *`var_name`* = *`value`*` | 全局 |
| `SET @@PERSIST.*`var_name`* = *`value`*` | 全局 |
| `SET PERSIST_ONLY *`var_name`* = *`value`*` | 无运行时效果 |
| `SET @@PERSIST_ONLY.*`var_name`* = *`value`*` | 无运行时效果 |
| `SET SESSION *`var_name`* = *`value`*` | 会话 |
| `SET @@SESSION.*`var_name`* = *`value`*` | 会话 |
| `SET *`var_name`* = *`value`*` | 会话 |
| `SET @@*`var_name`* = *`value`*` | 仅适用于下一个事务 |
| 语法 | 受影响的特征范围 |

可以在运行时检查事务特征的全局和会话值：

```sql
SELECT @@GLOBAL.transaction_isolation, @@GLOBAL.transaction_read_only;
SELECT @@SESSION.transaction_isolation, @@SESSION.transaction_read_only;
```
