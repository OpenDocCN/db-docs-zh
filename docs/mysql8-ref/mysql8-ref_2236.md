> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-sys-get-config.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-sys-get-config.html)

#### 30.4.5.19 sys_get_config()函数

给定配置选项名称，从`sys_config`表中返回选项值，如果该选项在表中不存在，则返回提供的默认值（可能为`NULL`）。

如果`sys_get_config()`函数")返回默认值且该值为`NULL`，则期望调用者能够处理给定配置选项的`NULL`。

按照惯例，调用`sys_get_config()`函数")的例程首先检查相应的用户定义变量是否存在且非`NULL`。如果是，则例程使用变量值而不读取`sys_config`表中的选项值。如果变量不存在或为`NULL`，则例程从表中读取选项值并将用户定义变量设置为该值。有关配置选项及其相应用户定义变量之间关系的更多信息，请参见第 30.4.2.1 节，“sys_config 表”。

如果要检查配置选项是否已设置，并在没有设置时使用`sys_get_config()`的返回值，可以使用`IFNULL(...)`（见后面的示例）。但是，这不应该在循环内执行（例如，在结果集中的每一行），因为对于需要在第一次迭代中才需要赋值的重复调用，使用`IFNULL(...)`预计比使用`IF (...) THEN ... END IF;`块要慢得多（见后面的示例）。

##### 参数

+   `in_variable_name VARCHAR(128)`: 要返回值的配置选项的名称。

+   `in_default_value VARCHAR(128)`: 如果在`sys_config`表中找不到配置选项，则返回的默认值。

##### 返回值

一个`VARCHAR(128)`值。

##### 示例

从`sys_config`表中获取配置值，如果该选项在表中不存在，则返回默认值 128：

```sql
mysql> SELECT sys.sys_get_config('statement_truncate_len', 128) AS Value;
+-------+
| Value |
+-------+
| 64    |
+-------+
```

一行示例：检查选项是否已设置；如果没有，则分配`IFNULL(...)`结果（使用`sys_config`表中的值）：

```sql
mysql> SET @sys.statement_truncate_len =
       IFNULL(@sys.statement_truncate_len,
              sys.sys_get_config('statement_truncate_len', 64));
```

`IF (...) THEN ... END IF;`块示例：检查选项是否已设置；如果没有，则分配`sys_config`表中的值：

```sql
IF (@sys.statement_truncate_len IS NULL) THEN
  SET @sys.statement_truncate_len = sys.sys_get_config('statement_truncate_len', 64);
END IF;
```
