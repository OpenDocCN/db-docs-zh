> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk-check-options.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-check-options.html)

#### 6.6.4.2 myisamchk 检查选项

**myisamchk**支持以下选项用于表检查操作：

+   `--check`, `-c`

    | 命令行格式 | `--check` |
    | --- | --- |

    检查表中的错误。如果未明确指定选择操作类型的选项，则这是默认操作。

+   `--check-only-changed`, `-C`

    | 命令行格式 | `--check-only-changed` |
    | --- | --- |

    仅检查自上次检查以来发生更改的表。

+   `--extend-check`, `-e`

    | 命令行格式 | `--extend-check` |
    | --- | --- |

    非常彻底地检查表。如果表具有许多索引，则这将非常慢。此选项仅应在极端情况下使用。通常情况下，**myisamchk**或**myisamchk --medium-check**应该能够确定表中是否存在任何错误。

    如果您使用`--extend-check`并且有足够的内存，将`key_buffer_size`变量设置为较大的值可以帮助修复操作运行更快。

    也请参阅表修复选项下此选项的描述。

    有关输出格式的描述，请参阅第 6.6.4.5 节，“使用 myisamchk 获取表信息”。

+   `--fast`, `-F`

    | 命令行格式 | `--fast` |
    | --- | --- |

    仅检查未正确关闭的表。

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    如果**myisamchk**在表中发现任何错误，则自动执行修复操作。修复类型与使用`--recover`或`-r`选项指定的相同。

+   `--information`, `-i`

    | 命令行格式 | `--information` |
    | --- | --- |

    打印有关所检查表的信息统计。

+   `--medium-check`, `-m`

    | 命令行格式 | `--medium-check` |
    | --- | --- |

    进行比`--extend-check`操作更快的检查。这只能找到 99.99%的所有错误，在大多数情况下应该足够好。

+   `--read-only`, `-T`

    | 命令行格式 | `--read-only` |
    | --- | --- |

    不要将表标记为已检查。如果您使用**myisamchk**来检查正在被某些不使用锁定的其他应用程序使用的表，比如当**mysqld**在外部锁定禁用时运行时，这将非常有用。

+   `--update-state`, `-U`

    | 命令行格式 | `--update-state` |
    | --- | --- |

    将信息存储在`.MYI`文件中，以指示表何时被检查以及表是否崩溃。这应该用于充分利用`--check-only-changed`选项的好处，但如果**mysqld**服务器正在使用该表并且您正在禁用外部锁定运行它，则不应使用此选项。
