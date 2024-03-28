> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-variable.html`](https://dev.mysql.com/doc/refman/8.0/en/set-variable.html)

#### 15.7.6.1 SET 变量赋值语法

```sql
SET *variable* = *expr* [, *variable* = *expr*] ...

*variable*: {
    *user_var_name*
  | *param_name*
  | *local_var_name*
  | {GLOBAL | @@GLOBAL.} *system_var_name*
  | {PERSIST | @@PERSIST.} *system_var_name*
  | {PERSIST_ONLY | @@PERSIST_ONLY.} *system_var_name*
  | [SESSION | @@SESSION. | @@] *system_var_name*
}
```

`SET`语法用于分配值给不同类型的变量，影响服务器或客户端的操作：

+   用户定义的变量。参见第 11.4 节，“用户定义的变量”。

+   存储过程和函数参数，以及存储程序局部变量。参见第 15.6.4 节，“存储程序中的变量”。

+   系统变量。参见第 7.1.8 节，“服务器系统变量”。系统变量也可以在服务器启动时设置，如第 7.1.9 节，“使用系统变量”所述。

一个`SET`语句分配变量值不会写入二进制日志，因此在复制场景中仅影响执行该语句的主机。要影响所有复制主机，请在每个主机上执行该语句。

以下各节描述了用于设置变量的`SET`语法。它们使用`=`赋值运算符，但`:=`赋值运算符也可用于此目的。

+   用户定义的变量赋值

+   参数和局部变量赋值

+   系统变量赋值

+   SET 错误处理

+   多变量赋值

+   表达式中的系统变量引用

##### 用户定义的变量赋值

用户定义的变量在会话内部局部创建，仅在该会话的上下文中存在；参见第 11.4 节，“用户定义的变量”。

用户定义的变量写作`@*`var_name`*`，并按以下方式分配表达式值：

```sql
SET @*var_name* = *expr*;
```

示例:

```sql
SET @name = 43;
SET @total_tax = (SELECT SUM(tax) FROM taxable_transactions);
```

正如这些语句所示，*`expr`*可以从简单（字面值）到更复杂（标量子查询返回的值）。

Performance Schema `user_variables_by_thread` 表包含有关用户定义变量的信息。请参阅 Section 29.12.10, “Performance Schema User-Defined Variable Tables”。

##### 参数和局部变量赋值

`SET` 适用于存储对象内部定义的参数和局部变量。以下过程使用了`increment`过程参数和`counter`局部变量：

```sql
CREATE PROCEDURE p(increment INT)
BEGIN
  DECLARE counter INT DEFAULT 0;
  WHILE counter < 10 DO
    -- ... do work ...
    SET counter = counter + increment;
  END WHILE;
END;
```

##### 系统变量赋值

MySQL 服务器维护配置其操作的系统变量。系统变量可以具有影响整个服务器操作的全局值，影响当前会话的会话值，或两者都有。许多系统变量是动态的，可以使用`SET`语句在运行时更改，以影响当前服务器实例的操作。`SET`还可以用于将某些系统变量持久化到数据目录中的`mysqld-auto.cnf`文件中，以影响后续启动的服务器操作。

如果对敏感系统变量发出`SET`语句，则在将其记录到一般日志和审计日志之前，查询将被重写以用“`<redacted>`”替换值。即使在服务器实例上没有通过密钥环组件进行安全存储，这也会发生。

如果更改会话系统变量，则该值在您的会话中保持有效，直到您将变量更改为不同的值或会话结束。更改对其他会话没有影响。

如果更改全局系统变量，则该值将被记住，并用于初始化新会话的会话值，直到您将变量更改为不同的值或服务器退出。更改对访问全局值的任何客户端都是可见的。但是，更改仅影响在更改后连接的客户端的相应会话值。全局变量更改不会影响任何当前客户端会话的会话值（甚至不会影响进行全局值更改的会话）。

要使全局系统变量设置永久生效，以便在服务器重新启动时应用，您可以将其持久化到数据目录中的`mysqld-auto.cnf`文件中。也可以通过手动修改`my.cnf`选项文件来进行持久化配置更改，但这样做更加繁琐，手动输入设置中的错误可能要等到很久之后才能发现。持久化系统变量的`SET`语句更加方便，避免了设置语法错误的可能性，因为具有语法错误的设置不会成功，也不会更改服务器配置。有关持久化系统变量和`mysqld-auto.cnf`文件的更多信息，请参见第 7.1.9.3 节，“持久化系统变量”。

注意

设置或持久化全局系统变量值始终需要特殊权限。通常设置会话系统变量值不需要特殊权限，任何用户都可以执行，尽管也有例外情况。有关更多信息，请参见第 7.1.9.1 节，“系统变量权限”。

以下讨论描述了设置和持久化系统变量的语法选项：

+   要为全局系统变量分配一个值，请在变量名称之前加上`GLOBAL`关键字或`@@GLOBAL.`修饰符：

    ```sql
    SET GLOBAL max_connections = 1000;
    SET @@GLOBAL.max_connections = 1000;
    ```

+   要为会话系统变量分配一个值，请在变量名称之前加上`SESSION`或`LOCAL`关键字，或者使用`@@SESSION.`、`@@LOCAL.`或`@@`修饰符，或者根本不使用关键字或修饰符：

    ```sql
    SET SESSION sql_mode = 'TRADITIONAL';
    SET LOCAL sql_mode = 'TRADITIONAL';
    SET @@SESSION.sql_mode = 'TRADITIONAL';
    SET @@LOCAL.sql_mode = 'TRADITIONAL';
    SET @@sql_mode = 'TRADITIONAL';
    SET sql_mode = 'TRADITIONAL';
    ```

    客户端可以更改自己的会话变量，但不能更改任何其他客户端的变量。

+   要将全局系统变量持久化到数据目录中的`mysqld-auto.cnf`选项文件中，请在变量名称之前加上`PERSIST`关键字或`@@PERSIST.`修饰符：

    ```sql
    SET PERSIST max_connections = 1000;
    SET @@PERSIST.max_connections = 1000;
    ```

    此`SET`语法允许您在运行时进行配置更改，这些更改也会在服务器重新启动时保留。与`SET GLOBAL`类似，`SET PERSIST`设置全局变量的运行时值，并将变量设置写入`mysqld-auto.cnf`文件（如果存在任何现有变量设置，则会替换）。

+   要将全局系统变量持久化到`mysqld-auto.cnf`文件中，而不设置全局变量的运行时值，请在变量名称之前加上`PERSIST_ONLY`关键字或`@@PERSIST_ONLY.`修饰符：

    ```sql
    SET PERSIST_ONLY back_log = 100;
    SET @@PERSIST_ONLY.back_log = 100;
    ```

    与`PERSIST`类似，`PERSIST_ONLY`将变量设置写入`mysqld-auto.cnf`。但是，与`PERSIST`不同，`PERSIST_ONLY`不会修改全局变量的运行时值。这使得`PERSIST_ONLY`适用于配置只能在服务器启动时设置的只读系统变量。

要将全局系统变量值设置为编译时 MySQL 默认值或会话系统变量设置为当前对应的全局值，将变量设置为值 `DEFAULT`。例如，以下两个语句在将 `max_join_size` 的会话值设置为当前全局值时是相同的：

```sql
SET @@SESSION.max_join_size = DEFAULT;
SET @@SESSION.max_join_size = @@GLOBAL.max_join_size;
```

使用 `SET` 来将全局系统变量持久化为 `DEFAULT` 值或其字面默认值，会将变量赋予其默认值并在 `mysqld-auto.cnf` 中添加变量的设置。要从文件中移除变量，使用 `RESET PERSIST`。

一些系统变量无法持久化或者受到持久化限制。参见第 7.1.9.4 节，“不可持久化和受限制持久化的系统变量”。

如果插件在执行 `SET` 语句时已安装，则插件实现的系统变量可以持久化。如果插件仍然安装，则持久化插件变量的赋值会在后续服务器重启时生效。如果插件不再安装，则当服务器读取 `mysqld-auto.cnf` 文件时，插件变量将不再存在。在这种情况下，服务器会向错误日志写入警告并继续：

```sql
currently unknown variable '*var_name*'
was read from the persisted config file
```

要显示系统变量名称和值：

+   使用 `SHOW VARIABLES` 语句；参见第 15.7.7.41 节，“SHOW VARIABLES 语句”。

+   几个 Performance Schema 表提供系统变量信息。参见第 29.12.14 节，“Performance Schema System Variable Tables”。

+   Performance Schema `variables_info` 表包含了显示每个系统变量最近由哪个用户何时设置的信息。参见第 29.12.14.2 节，“Performance Schema variables_info Table”。

+   Performance Schema `persisted_variables` 表提供了一个 SQL 接口来访问 `mysqld-auto.cnf` 文件，使得可以在运行时使用 `SELECT` 语句检查其内容。参见第 29.12.14.1 节，“Performance Schema persisted_variables Table”。

##### 设置错误处理

如果`SET`语句中的任何变量赋值失败，则整个语句失败，变量不会更改，`mysqld-auto.cnf`文件也不会更改。

`SET`在这里描述的情况下会产生错误。大多数示例显示使用关键字语法的`SET`语句（例如，`GLOBAL`或`SESSION`），但这些原则也适用于使用相应修饰符的语句（例如，`@@GLOBAL.`或`@@SESSION.`）。

+   使用`SET`（任何变体）设置只读变量：

    ```sql
    mysql> SET GLOBAL version = 'abc';
    ERROR 1238 (HY000): Variable 'version' is a read only variable
    ```

+   使用`GLOBAL`、`PERSIST`或`PERSIST_ONLY`设置仅具有会话值的变量：

    ```sql
    mysql> SET GLOBAL sql_log_bin = ON;
    ERROR 1228 (HY000): Variable 'sql_log_bin' is a SESSION
    variable and can't be used with SET GLOBAL
    ```

+   使用`SESSION`设置仅具有全局值的变量：

    ```sql
    mysql> SET SESSION max_connections = 1000;
    ERROR 1229 (HY000): Variable 'max_connections' is a
    GLOBAL variable and should be set with SET GLOBAL
    ```

+   省略`GLOBAL`、`PERSIST`或`PERSIST_ONLY`以设置仅具有全局值的变量：

    ```sql
    mysql> SET max_connections = 1000;
    ERROR 1229 (HY000): Variable 'max_connections' is a
    GLOBAL variable and should be set with SET GLOBAL
    ```

+   使用`PERSIST`或`PERSIST_ONLY`设置无法持久化的变量：

    ```sql
    mysql> SET PERSIST port = 3307;
    ERROR 1238 (HY000): Variable 'port' is a read only variable
    mysql> SET PERSIST_ONLY port = 3307;
    ERROR 1238 (HY000): Variable 'port' is a non persistent read only variable
    ```

+   `@@GLOBAL.`、`@@PERSIST.`、`@@PERSIST_ONLY.`、`@@SESSION.`和`@@`修饰符仅适用于系统变量。尝试将它们应用于用户定义的变量、存储过程或函数参数或存储程序本地变量会导致错误。

+   并非所有系统变量都可以设置为`DEFAULT`。在这种情况下，分配`DEFAULT`会导致错误。

+   尝试将`DEFAULT`分配给用户定义的变量、存储过程或函数参数或存储程序本地变量会导致错误。

##### 多变量赋值

一个`SET`语句可以包含多个变量赋值，用逗号分隔。此语句将值分配给用户定义的变量和系统变量：

```sql
SET @x = 1, SESSION sql_mode = '';
```

如果在单个语句中设置多个系统变量，则该语句中最近的`GLOBAL`、`PERSIST`、`PERSIST_ONLY`或`SESSION`关键字用于后续未指定关键字的赋值。

多变量赋值的示例：

```sql
SET GLOBAL sort_buffer_size = 1000000, SESSION sort_buffer_size = 1000000;
SET @@GLOBAL.sort_buffer_size = 1000000, @@LOCAL.sort_buffer_size = 1000000;
SET GLOBAL max_connections = 1000, sort_buffer_size = 1000000;
```

`@@GLOBAL.`、`@@PERSIST.`、`@@PERSIST_ONLY.`、`@@SESSION.`和`@@`修饰符仅适用于紧接着的系统变量，而不适用于任何剩余的系统变量。此语句将`sort_buffer_size`全局值设置为 50000，会话值设置为 1000000：

```sql
SET @@GLOBAL.sort_buffer_size = 50000, sort_buffer_size = 1000000;
```

##### 表达式中的系统变量引用

要在表达式中引用系统变量的值，请使用`@@`修饰符之一（除了在表达式中不允许使用`@@PERSIST.`和`@@PERSIST_ONLY.`）。例如，您可以在`SELECT`语句中像这样检索系统变量的值：

```sql
SELECT @@GLOBAL.sql_mode, @@SESSION.sql_mode, @@sql_mode;
```

注意

在表达式中引用系统变量作为`@@*`var_name`*`（使用`@@`而不是`@@GLOBAL.`或`@@SESSION.`）如果存在会返回会话值，否则返回全局值。这与`SET @@*`var_name`* = *`expr`*`不同，后者始终引用会话值。
