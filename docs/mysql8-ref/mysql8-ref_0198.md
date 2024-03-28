> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk-general-options.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-general-options.html)

#### 6.6.4.1 myisamchk 通用选项

本节描述的选项可用于由**myisamchk**执行的任何类型的表维护操作。本节之后的部分描述了仅适用于特定操作（如表检查或修复）的选项。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。选项按操作类型分组。

+   `--HELP`, `-H`

    | 命令行格式 | `--HELP` |
    | --- | --- |

    显示帮助消息并退出。选项以单个列表的形式呈现。

+   `--debug=*`debug_options`*`, `-# *`debug_options`*`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o,/tmp/myisamchk.trace` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认值为`d:t:o,/tmp/myisamchk.trace`。

    此选项仅在使用`WITH_DEBUG`构建 MySQL 时可用。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--defaults-extra-file=*`file_name`*`

    | 命令行格式 | `--defaults-extra-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    在全局选项文件之后但（在 Unix 上）在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此及其他选项文件选项的附加信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-file=*`file_name`*`

    | 命令行格式 | `--defaults-file=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    仅使用给定的选项文件。如果文件不存在或无法访问，将会出现错误。如果*`file_name`*不是绝对路径名，则将其解释为相对于当前目录。

    有关此及其他选项文件选项的附加信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--defaults-group-suffix=*`str`*`

    | 命令行格式 | `--defaults-group-suffix=str` |
    | --- | --- |
    | 类型 | 字符串 |

    不仅读取通常的选项组，还读取具有通常名称和后缀*`str`*的组。例如，**myisamchk**通常会读取`[myisamchk]`组。如果将此选项指定为`--defaults-group-suffix=_other`，**myisamchk**还会读取`[myisamchk_other]`组。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--no-defaults`

    | 命令行格式 | `--no-defaults` |
    | --- | --- |

    不要读取任何选项文件。如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用`--no-defaults`来防止读取它们。

    例外情况是，如果存在`.mylogin.cnf`文件，则在所有情况下都会读取该文件。即使使用`--no-defaults`，也可以以比在命令行上更安全的方式指定密码。要创建`.mylogin.cnf`，请使用**mysql_config_editor**实用程序。请参阅第 6.6.7 节，“mysql_config_editor — MySQL 配置实用程序”。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--print-defaults`

    | 命令行格式 | `--print-defaults` |
    | --- | --- |

    打印���序名称和从选项文件获取的所有选项。

    有关此选项和其他选项文件选项的其他信息，请参阅第 6.2.2.3 节，“影响选项文件处理的命令行选项”。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    静默模式。仅在发生错误时才写入输出。您可以两次使用`-s`（`-ss`）使**myisamchk**非常安静。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多信息。可以与`-d`和`-e`一起使用。多次使用`-v`（`-vv`，`-vvv`）以获得更多输出。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--wait`，`-w`

    | 命令行格式 | `--wait` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    在表被锁定时不会终止，而是等待表解锁后继续。如果你正在运行**mysqld**并禁用外部锁定，那么表只能被另一个**myisamchk**命令锁定。

你也可以使用`--*`var_name`*=*`value`*`语法设置以下变量：

| 变量 | 默认值 |
| --- | --- |
| `decode_bits` | 9 |
| `ft_max_word_len` | 版本相关 |
| `ft_min_word_len` | 4 |
| `ft_stopword_file` | 内置列表 |
| `key_buffer_size` | 523264 |
| `myisam_block_size` | 1024 |
| `myisam_sort_key_blocks` | 16 |
| `read_buffer_size` | 262136 |
| `sort_buffer_size` | 2097144 |
| `sort_key_blocks` | 16 |
| `stats_method` | nulls_unequal |
| `write_buffer_size` | 262136 |
| 变量 | 默认值 |

可以使用**myisamchk --help**查看可能的**myisamchk**变量及其默认值：

当通过排序键修复键时，会使用`myisam_sort_buffer_size`，这是在使用`--recover`时的正常情况。`sort_buffer_size`是`myisam_sort_buffer_size`的弃用同义词。

当你使用`--extend-check`检查表或通过逐行将键插入表中修复键时（例如进行正常插入时），会使用`key_buffer_size`。通过键缓冲区进行修复在以下情况下使用：

+   你使用`--safe-recover`。

+   为了对键进行排序所需的临时文件大小会比直接创建键文件时大两倍以上。当你有大型键值的`CHAR`、`VARCHAR`或`TEXT`列时，通常会出现这种情况，因为排序操作需要在进行过程中存储完整的键值。如果你有大量临时空间，并且可以强制**myisamchk**通过排序来修复，你可以使用`--sort-recover`选项。

通过键缓冲区进行修复比使用排序需要的磁盘空间少得多，但速度也慢得多。

如果您想要更快的修复，请将`key_buffer_size`和`myisam_sort_buffer_size`变量设置为可用内存的约 25%。您可以将这两个变量都设置为较大的值，因为一次只使用其中一个。

`myisam_block_size`是用于索引块的大小。

`stats_method`影响在给定`--analyze`选项时如何处理`NULL`值以进行索引统计收集。它的作用类似于`myisam_stats_method`系统变量。有关更多信息，请参阅第 7.1.8 节，“服务器系统变量”中`myisam_stats_method`的描述，以及第 10.3.8 节，“InnoDB 和 MyISAM 索引统计收集”。

`ft_min_word_len`和`ft_max_word_len`表示`MyISAM`表上`FULLTEXT`索引的最小和最大单词长度。`ft_stopword_file`指定停用词文件。这些需要在以下情况下设置。

如果您使用**myisamchk**执行修改表索引的操作（如修复或分析），则`FULLTEXT`索引将使用默认的全文参数值进行重建，除非您另有指定。否则可能导致查询失败。

问题出现在这些参数只有服务器知道。它们不存储在`MyISAM`索引文件中。如果您在服务器中修改了最小或最大单词长度或停用词文件以避免问题，请为**myisamchk**指定与您用于**mysqld**相同的`ft_min_word_len`、`ft_max_word_len`和`ft_stopword_file`值。例如，如果您将最小单词长度设置为 3，您可以像这样使用**myisamchk**修复表：

```sql
myisamchk --recover --ft_min_word_len=3 *tbl_name*.MYI
```

为了确保**myisamchk**和服务器使用相同的全文参数值，您可以将每个值放在选项文件的`[mysqld]`和`[myisamchk]`部分中：

```sql
[mysqld]
ft_min_word_len=3

[myisamchk]
ft_min_word_len=3
```

使用`REPAIR TABLE`、`ANALYZE TABLE`、`OPTIMIZE TABLE`或`ALTER TABLE`是使用**myisamchk**的替代方法。这些语句由服务器执行，服务器知道要使用的正确全文参数值。
