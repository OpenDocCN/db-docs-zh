# 6.7.2 my_print_defaults — 从选项文件显示选项

> 原文：[`dev.mysql.com/doc/refman/8.0/en/my-print-defaults.html`](https://dev.mysql.com/doc/refman/8.0/en/my-print-defaults.html)

**my_print_defaults**显示选项文件中选项组中存在的选项。输出指示哪些选项被读取指定选项组的程序使用。例如，**mysqlcheck**程序读取`[mysqlcheck]`和`[client]`选项组。要查看标准选项文件中这些组中存在哪些选项，请像这样调用**my_print_defaults**：

```sql
$> my_print_defaults mysqlcheck client
--user=myusername
--password=*password*
--host=localhost
```

输出包含选项，每行一个，以它们在命令行上指定的形式。

**my_print_defaults**支持以下选项。

+   `--help`, `-?`

    显示帮助消息并退出。

+   `--config-file=*`file_name`*`, `--defaults-file=*`file_name`*`, `-c *`file_name`*`

    仅读取给定的选项文件。

+   `--debug=*`debug_options`*`, `-# *`debug_options`*`

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o,/tmp/my_print_defaults.trace`。

+   `--defaults-extra-file=*`file_name`*`, `--extra-file=*`file_name`*`, `-e *`file_name`*`

    在全局选项文件之后但在用户选项文件之前（在 Unix 上）读取此选项文件。

    有关此选项和其他选项文件选项的其他信息，请参见 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`suffix`*`, `-g *`suffix`*`

    除了在命令行上命名的组之外，还读取具有给定后缀的组。

    有关此选项和其他选项文件选项的其他信息，请参见 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--login-path=*`name`*`, `-l *`name`*`

    从`.mylogin.cnf`登录路径文件中的指定登录路径读取选项。 “登录路径”是一个包含指定要连接到哪个 MySQL 服务器以及要作为哪个帐户进行身份验证的选项的选项组。要创建或修改登录路径文件，请使用**mysql_config_editor**实用程序。请参见第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-defaults`, `-n`

    返回空字符串。

    有关此选项和其他选项文件选项的更多信息，请参见第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--show`, `-s`

    **my_print_defaults**默认情况下屏蔽密码。使用此选项以明文显示密码。

+   `--verbose`, `-v`

    冗长模式。打印程序执行的更多信息。

+   `--version`, `-V`

    显示版本信息并退出。
