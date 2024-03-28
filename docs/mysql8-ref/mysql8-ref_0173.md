# 6.4.1 comp_err — 编译 MySQL 错误消息文件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/comp-err.html`](https://dev.mysql.com/doc/refman/8.0/en/comp-err.html)

**comp_err** 创建了 `errmsg.sys` 文件，该文件被 **mysqld** 用来确定不同错误代码的错误消息显示。**comp_err** 通常在构建 MySQL 时会自动运行。它从 MySQL 源代码分发中的文本格式错误信息编译 `errmsg.sys` 文件：

+   截至 MySQL 8.0.19，错误信息来自 `share` 目录中的 `messages_to_error_log.txt` 和 `messages_to_clients.txt` 文件。

    要了解更多关于定义错误消息的信息，请参阅这些文件中的注释，以及 `errmsg_readme.txt` 文件。

+   在 MySQL 8.0.19 之前，错误信息来自 `sql/share` 目录中的 `errmsg-utf8.txt` 文件。

**comp_err** 还生成 `mysqld_error.h`、`mysqld_ername.h` 和 `mysqld_errmsg.h` 头文件。

调用 **comp_err** 如下：

```sql
comp_err [*options*]
```

**comp_err** 支持以下选项。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `false` |

    显示帮助消息并退出。

+   `--charset=*`dir_name`*`, `-C *`dir_name`*`

    | 命令行格式 | `--charset` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `../share/charsets` |

    字符集目录。默认为 `../sql/share/charsets`。

+   `--debug=*`debug_options`*`, `-# *`debug_options`*`

    | 命令行格式 | `--debug=options` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:O,/tmp/comp_err.trace` |

    写入调试日志。典型的 *`debug_options`* 字符串是 `d:t:O,*`file_name`*`。默认为 `d:t:O,/tmp/comp_err.trace`。

+   `--debug-info`, `-T`

    | 命令行格式 | `--debug-info` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `false` |

    程序退出时打印一些调试信息。

+   `--errmsg-file=*`file_name`*`, `-H *`file_name`*`

    | 命令行格式 | `--errmsg-file=name` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `mysqld_errmsg.h` |

    错误消息文件的名称。默认为 `mysqld_errmsg.h`。此选项在 MySQL 8.0.18 中添加。

+   `--header-file=*`file_name`*`, `-H *`file_name`*`

    | 命令行格式 | `--header-file=name` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `mysqld_error.h` |

    错误头文件的名称。默认为 `mysqld_error.h`。

+   `--in-file=*`file_name`*`, `-F *`file_name`*`

    | 命令行格式 | `--in-file=path` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    输入文件的名称。默认值为 `../share/errmsg-utf8.txt`。

    此选项在 MySQL 8.0.19 中被移除，并由 `--in-file-errlog` 和 `--in-file-toclient` 选项替代。

+   `--in-file-errlog=*`file_name`*`, `-e *`file_name`*`

    | 命令行格式 | `--in-file-errlog` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `../share/messages_to_error_log.txt` |

    定义要写入错误日志的错误消息的输入文件名。默认值为 `../share/messages_to_error_log.txt`。

    此选项在 MySQL 8.0.19 中添加。

+   `--in-file-toclient=*`file_name`*`, `-c *`file_name`*`

    | 命令行格式 | `--in-file-toclient=path` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `../share/messages_to_clients.txt` |

    定义要写入客户端的错误消息的输入文件名。默认值为 `../share/messages_to_clients.txt`。

    此选项在 MySQL 8.0.19 中添加。

+   `--name-file=*`file_name`*`, `-N *`file_name`*`

    | 命令行格式 | `--name-file=name` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `mysqld_ername.h` |

    错误名称文件的名称。默认值为 `mysqld_ername.h`。

+   `--out-dir=*`dir_name`*`, `-D *`dir_name`*`

    | 命令行格式 | `--out-dir=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `../share/` |

    输出基本目录的名称。默认值为 `../sql/share/`。

+   `--out-file=*`file_name`*`, `-O *`file_name`*`

    | 命令行格式 | `--out-file=name` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `errmsg.sys` |

    输出文件的名称。默认值为 `errmsg.sys`。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示版本信息并退出。
