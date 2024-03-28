> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-variables-info-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-variables-info-table.html)

#### 29.12.14.2 性能模式 variables_info 表

`variables_info`表显示了每个系统变量最近设置的来源以及其值范围。

`variables_info`表具有以下列：

+   `VARIABLE_NAME`

    变量名称。

+   `VARIABLE_SOURCE`

    变量最近设置的来源：

    +   `COMMAND_LINE`

        变量是从命令行设置的。

    +   `COMPILED`

        变量具有其编译默认值。`COMPILED`是未以其他方式设置变量的值。

    +   `DYNAMIC`

        变量是在运行时设置的。这包括使用`init_file`系统变量指定的文件中设置的变量。

    +   `EXPLICIT`

        变量是从一个名为`--defaults-file`的选项文件设置的。

    +   `EXTRA`

        变量是从一个名为`--defaults-extra-file`的选项文件设置的。

    +   `GLOBAL`

        变量是从全局选项文件设置的。这包括未被`EXPLICIT`, `EXTRA`, `LOGIN`, `PERSISTED`, `SERVER`, 或 `USER`覆盖的选项文件。

    +   `LOGIN`

        变量是从用户特定的登录路径文件(`~/.mylogin.cnf`)设置的。

    +   `PERSISTED`

        变量是从服务器特定的`mysqld-auto.cnf`选项文件设置的。如果服务器启动时禁用了`persisted_globals_load`，则没有行具有此值。

    +   `SERVER`

        变量是从服务器特定的``$MYSQL_HOME`/my.cnf`选项文件设置的。有关如何设置`MYSQL_HOME`的详细信息，请参见 Section 6.2.2.2, “使用选项文件”。

    +   `USER`

        变量是从用户特定的`~/.my.cnf`选项文件设置的。

+   `VARIABLE_PATH`

    如果变量是从选项文件设置的，则`VARIABLE_PATH`是该文件的路径名。否则，该值为空字符串。

+   `MIN_VALUE`, `MAX_VALUE`

    变量的最小和最大允许值。对于没有这些值的变量（即非数字变量），两者都为 0。

+   `SET_TIME`

    变量最近设置的时间。默认值是服务器在启动期间初始化全局系统变量的时间。

+   `SET_USER`, `SET_HOST`

    最近设置变量的客户端用户的用户名和主机名。如果客户端以`user17`从主机`host34.example.com`连接，使用帐户`'user17'@'%.example.com'`，则`SET_USER`和`SET_HOST`分别为`user17`和`host34.example.com`。对于代理用户连接，这些值对应于外部（代理）用户，而不是执行权限检查的被代理用户。每列的默认值为空字符串，表示自服务器启动以来未设置变量。

`variables_info` 表没有索引。

`TRUNCATE TABLE` 不允许用于`variables_info`表。

如果在运行时设置了具有`VARIABLE_SOURCE`值为`DYNAMIC`以外的变量，则`VARIABLE_SOURCE`变为`DYNAMIC`，而`VARIABLE_PATH`变为空字符串。

仅具有会话值的系统变量（例如`debug_sync`）无法在启动时或持久化设置。对于仅会话系统变量，`VARIABLE_SOURCE`只能是`COMPILED`或`DYNAMIC`。

如果系统变量具有意外的`VARIABLE_SOURCE`值，请考虑您的服务器启动方法。例如，**mysqld_safe** 读取选项文件，并将在那里找到的某些选项作为启动**mysqld**时使用的命令行的一部分传递。因此，您在选项文件中设置的一些系统变量可能会显示在`variables_info`中，而不是像您可能期望的那样显示为`COMMAND_LINE`，而不是`GLOBAL`或`SERVER`。

使用`variables_info`表的一些示例查询，以及代表性输出：

+   显示在命令行上设置的变量：

    ```sql
    mysql> SELECT VARIABLE_NAME
           FROM performance_schema.variables_info
           WHERE VARIABLE_SOURCE = 'COMMAND_LINE'
           ORDER BY VARIABLE_NAME;
    +---------------+
    | VARIABLE_NAME |
    +---------------+
    | basedir       |
    | datadir       |
    | log_error     |
    | pid_file      |
    | plugin_dir    |
    | port          |
    +---------------+
    ```

+   显示从持久存储设置的变量：

    ```sql
    mysql> SELECT VARIABLE_NAME
           FROM performance_schema.variables_info
           WHERE VARIABLE_SOURCE = 'PERSISTED'
           ORDER BY VARIABLE_NAME;
    +--------------------------+
    | VARIABLE_NAME            |
    +--------------------------+
    | event_scheduler          |
    | max_connections          |
    | validate_password.policy |
    +--------------------------+
    ```

+   将`variables_info`与`global_variables`表连接，以显示持久化变量的当前值以及其值范围：

    ```sql
    mysql> SELECT
             VI.VARIABLE_NAME, GV.VARIABLE_VALUE,
             VI.MIN_VALUE,VI.MAX_VALUE
           FROM performance_schema.variables_info AS VI
             INNER JOIN performance_schema.global_variables AS GV
             USING(VARIABLE_NAME)
           WHERE VI.VARIABLE_SOURCE = 'PERSISTED'
           ORDER BY VARIABLE_NAME;
    +--------------------------+----------------+-----------+-----------+
    | VARIABLE_NAME            | VARIABLE_VALUE | MIN_VALUE | MAX_VALUE |
    +--------------------------+----------------+-----------+-----------+
    | event_scheduler          | ON             | 0         | 0         |
    | max_connections          | 200            | 1         | 100000    |
    | validate_password.policy | STRONG         | 0         | 0         |
    +--------------------------+----------------+-----------+-----------+
    ```
