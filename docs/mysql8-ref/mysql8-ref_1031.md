> 原文：[`dev.mysql.com/doc/refman/8.0/en/declare-condition.html`](https://dev.mysql.com/doc/refman/8.0/en/declare-condition.html)

#### 15.6.7.1 DECLARE ... CONDITION Statement

```sql
DECLARE *condition_name* CONDITION FOR *condition_value*

*condition_value*: {
    *mysql_error_code*
  | SQLSTATE [VALUE] *sqlstate_value*
}
```

`DECLARE ... CONDITION` 语句声明了一个命名的错误条件，将一个名称与需要特定处理的条件关联起来。该名称可以在随后的 `DECLARE ... HANDLER` 语句中引用（参见 Section 15.6.7.2, “DECLARE ... HANDLER Statement”）。

条件声明必须出现在游标或处理程序声明之前。

`DECLARE ... CONDITION` 的 *`condition_value`* 指示与条件名称关联的特定条件或条件类。它可以采用以下形式：

+   *`mysql_error_code`*: 表示 MySQL 错误代码的整数文字。

    不要使用 MySQL 错误代码 0，因为这表示成功而不是错误条件。有关 MySQL 错误代码的列表，请参见 Server Error Message Reference。

+   SQLSTATE [VALUE] *`sqlstate_value`*: 一个表示 SQLSTATE 值的 5 个字符的字符串文字。

    不要使用以 `'00'` 开头的 SQLSTATE 值，因为这些值表示成功而不是错误条件。有关 SQLSTATE 值的列表，请参见 Server Error Message Reference。

在 `SIGNAL` 或使用 `RESIGNAL` 语句中引用的条件名称必须与 SQLSTATE 值关联，而不是 MySQL 错误代码。

使用条件名称可以帮助使存储过程代码更清晰。例如，此处理程序适用于尝试删除不存在的表，但只有当您知道 1051 是 MySQL 错误代码“未知表”时才明显：

```sql
DECLARE CONTINUE HANDLER FOR 1051
  BEGIN
    -- body of handler
  END;
```

通过为条件声明一个名称，处理程序的目的更容易看到：

```sql
DECLARE no_such_table CONDITION FOR 1051;
DECLARE CONTINUE HANDLER FOR no_such_table
  BEGIN
    -- body of handler
  END;
```

这里有一个命名条件，与相同条件相对应，但基于相应的 SQLSTATE 值而不是 MySQL 错误代码：

```sql
DECLARE no_such_table CONDITION FOR SQLSTATE '42S02';
DECLARE CONTINUE HANDLER FOR no_such_table
  BEGIN
    -- body of handler
  END;
```
