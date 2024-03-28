# 6.6.5 myisamlog — 显示 MyISAM 日志文件内容

> [`dev.mysql.com/doc/refman/8.0/en/myisamlog.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamlog.html)

**myisamlog** 处理 `MyISAM` 日志文件的内容。要创建这样一个文件，请使用 `--log-isam=`log_file`` 选项启动服务器。

像这样调用 **myisamlog**：

```sql
myisamlog [*options*] [*file_name* [*tbl_name*] ...]
```

默认操作是更新 (`-u`)。如果执行恢复操作 (`-r`)，则会执行所有写入操作，可能还包括更新和删除操作，并且只计算错误。如果没有给出 *`log_file`* 参数，则默认日志文件名为 `myisam.log`。如果在命令行上命名了表，则只更新这些表。

**myisamlog** 支持以下选项：

+   `-?`, `-I`

    显示帮助信息并退出。

+   `-c *`N`*`

    仅执行 *`N`* 条命令。

+   `-f *`N`*`

    指定最大打开文件数。

+   `-F *`filepath/`*`

    指定带有尾随斜杠的文件路径。

+   `-i`

    在退出之前显示额外信息。

+   `-o *`offset`*`

    指定起始偏移量。

+   `-p *`N`*`

    从路径中移除 *`N`* 个组件。

+   `-r`

    执行恢复操作。

+   `-R *`record_pos_file record_pos`*`

    指定记录位置文件和记录位置。

+   `-u`

    执行更新操作。

+   `-v`

    详细模式。打印有关程序操作的更多输出。可以多次使用此选项以产生更多输出。

+   `-w *`write_file`*`

    指定写入文件。

+   `-V`

    显示版本信息。
