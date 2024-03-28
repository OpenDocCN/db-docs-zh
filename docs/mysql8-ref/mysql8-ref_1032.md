> 原文：[`dev.mysql.com/doc/refman/8.0/en/declare-handler.html`](https://dev.mysql.com/doc/refman/8.0/en/declare-handler.html)

#### 15.6.7.2 DECLARE ... HANDLER Statement

```sql
DECLARE *handler_action* HANDLER
    FOR *condition_value* [, *condition_value*] ...
    *statement*

*handler_action*: {
    CONTINUE
  | EXIT
  | UNDO
}

*condition_value*: {
    *mysql_error_code*
  | SQLSTATE [VALUE] *sqlstate_value*
  | *condition_name*
  | SQLWARNING
  | NOT FOUND
  | SQLEXCEPTION
}
```

`DECLARE ... HANDLER` 语句指定处理一个或多个条件的处理程序。如果其中一个条件发生，则执行指定的 *`statement`*。*`statement`* 可以是一个简单语句，如 `SET *`var_name`* = *`value`*`，或者使用 `BEGIN` 和 `END` 编写的复合语句（参见 Section 15.6.1, “BEGIN ... END Compound Statement”）。

处理程序声明必须出现在变量或条件声明之后。

*`handler_action`* 值表示处理程序在执行处理程序语句后采取的操作：

+   `CONTINUE`：当前程序的执行继续。

+   `EXIT`：执行终止于声明处理程序的 `BEGIN ... END` 复合语句。即使条件发生在内部块中，这也是正确的。

+   `UNDO`：不支持。

`DECLARE ... HANDLER` 的 *`condition_value`* 指示激活处理程序的特定条件或条件类别。它可以采用以下形式：

+   *`mysql_error_code`*：表示 MySQL 错误代码的整数文字，例如 1051 表示“未知表”：

    ```sql
    DECLARE CONTINUE HANDLER FOR 1051
      BEGIN
        -- body of handler
      END;
    ```

    不要使用 MySQL 错误代码 0，因为这表示成功而不是错误条件。有关 MySQL 错误代码的列表，请参阅 Server Error Message Reference。

+   SQLSTATE [VALUE] *`sqlstate_value`*：表示 SQLSTATE 值的 5 个字符字符串文字，例如 `'42S01'` 表示“未知表”：

    ```sql
    DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02'
      BEGIN
        -- body of handler
      END;
    ```

    不要使用以 `'00'` 开头的 SQLSTATE 值，因为这些值表示成功而不是错误条件。有关 SQLSTATE 值的列表，请参阅 Server Error Message Reference。

+   *`condition_name`*：先前使用 `DECLARE ... CONDITION` 指定的条件名。条件名可以与 MySQL 错误代码或 SQLSTATE 值关联。请参阅 Section 15.6.7.1, “DECLARE ... CONDITION Statement”。

+   `SQLWARNING`：简写为以 `'01'` 开头的 SQLSTATE 值类别。

    ```sql
    DECLARE CONTINUE HANDLER FOR SQLWARNING
      BEGIN
        -- body of handler
      END;
    ```

+   `NOT FOUND`：简写为以 `'02'` 开头的 SQLSTATE 值类别。在游标的上下文中相关，并用于控制当游标到达数据集末尾时发生的情况。如果没有更多行可用，则会发生无数据条件，其 SQLSTATE 值为 `'02000'`。要检测此条件，可以为其或为 `NOT FOUND` 条件设置处理程序。

    ```sql
    DECLARE CONTINUE HANDLER FOR NOT FOUND
      BEGIN
        -- body of handler
      END;
    ```

    举个例子，参见第 15.6.6 节“游标”。`NOT FOUND`条件也适用于检索不到行的`SELECT ... INTO *var_list*`语句。

+   `SQLEXCEPTION`：SQLSTATE 值的类别的简写，这些值不以`'00'`、`'01'`或`'02'`开头。

    ```sql
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
      BEGIN
        -- body of handler
      END;
    ```

有关服务器在条件发生时选择处理程序的信息，请参见第 15.6.7.6 节“处理程序的作用域规则”。

如果发生未声明处理程序的条件，则采取的操作取决于条件类别：

+   对于`SQLEXCEPTION`条件，存储过程在引发条件的语句处终止，就好像有一个`EXIT`处理程序一样。如果程序是由另一个存储过程调用的，则调用程序使用处理程序选择规则处理条件。

+   对于`SQLWARNING`条件，程序会继续执行，就好像有一个`CONTINUE`处理程序一样。

+   对于`NOT FOUND`条件，如果条件正常引发，操作是`CONTINUE`。如果是由`SIGNAL`或`RESIGNAL`引发的，操作是`EXIT`。

以下示例使用了一个`SQLSTATE '23000'`的处理程序，该处理程序用于处理重复键错误：

```sql
mysql> CREATE TABLE test.t (s1 INT, PRIMARY KEY (s1));
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter //

mysql> CREATE PROCEDURE handlerdemo ()
       BEGIN
         DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' SET @x2 = 1;
         SET @x = 1;
         INSERT INTO test.t VALUES (1);
         SET @x = 2;
         INSERT INTO test.t VALUES (1);
         SET @x = 3;
       END;
       //
Query OK, 0 rows affected (0.00 sec)

mysql> CALL handlerdemo()//
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @x//
 +------+
 | @x   |
 +------+
 | 3    |
 +------+
    1 row in set (0.00 sec)
```

注意，在过程执行后，`@x`为`3`，这表明在错误发生后，执行继续到过程结束。如果`DECLARE ... HANDLER`语句不存在，MySQL 会在第二个由于`PRIMARY KEY`约束而导致的`INSERT`失败后采取默认操作（`EXIT`），`SELECT @x`将返回`2`。

要忽略一个条件，为其声明一个`CONTINUE`处理程序，并将其与一个空块关联。例如：

```sql
DECLARE CONTINUE HANDLER FOR SQLWARNING BEGIN END;
```

块标签的作用域不包括在块内声明的处理程序的代码。因此，与处理程序关联的语句不能使用`ITERATE`或`LEAVE`来引用包围处理程序声明的块的标签。考虑以下示例，其中`REPEAT`块具有`retry`标签：

```sql
CREATE PROCEDURE p ()
BEGIN
  DECLARE i INT DEFAULT 3;
  retry:
    REPEAT
      BEGIN
        DECLARE CONTINUE HANDLER FOR SQLWARNING
          BEGIN
            ITERATE retry;    # illegal
          END;
        IF i < 0 THEN
          LEAVE retry;        # legal
        END IF;
        SET i = i - 1;
      END;
    UNTIL FALSE END REPEAT;
END;
```

`retry`标签在块内的`IF`语句中有效。它对于`CONTINUE`处理程序无效，因此那里的引用是无效的，会导致错误：

```sql
ERROR 1308 (42000): LEAVE with no matching label: retry
```

为了避免在处理程序中引用外部标签，可以使用以下策略之一：

+   要离开块，请使用`EXIT`处理程序。如果不需要块清理，`BEGIN ... END`处理程序主体可以为空：

    ```sql
    DECLARE EXIT HANDLER FOR SQLWARNING BEGIN END;
    ```

    否则，在处理程序主体中放置清理语句：

    ```sql
    DECLARE EXIT HANDLER FOR SQLWARNING
      BEGIN
        *block cleanup statements*
      END;
    ```

+   要继续执行，请在`CONTINUE`处理程序中设置一个状态变量，该变量可以在封闭块中进行检查，以确定处理程序是否被调用。以下示例使用变量`done`来实现这一目的：

    ```sql
    CREATE PROCEDURE p ()
    BEGIN
      DECLARE i INT DEFAULT 3;
      DECLARE done INT DEFAULT FALSE;
      retry:
        REPEAT
          BEGIN
            DECLARE CONTINUE HANDLER FOR SQLWARNING
              BEGIN
                SET done = TRUE;
              END;
            IF done OR i < 0 THEN
              LEAVE retry;
            END IF;
            SET i = i - 1;
          END;
        UNTIL FALSE END REPEAT;
    END;
    ```
