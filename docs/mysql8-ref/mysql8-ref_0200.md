> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk-repair-options.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-repair-options.html)

#### 6.6.4.3 myisamchk 修复选项

**myisamchk**支持以下选项用于表修复操作（在给定诸如`--recover`或`--safe-recover`之类的选项时执行的操作）：

+   `--backup`, `-B`

    | 命令行格式 | `--backup` |
    | --- | --- |

    将`.MYD`文件备份为`*`file_name`*-*`time`*.BAK`

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    安装字符集的目录。请参见第 12.15 节，“字符集配置”。

+   `--correct-checksum`

    | 命令行格式 | `--correct-checksum` |
    | --- | --- |

    为表正确校验和信息。

+   `--data-file-length=*`len`*`, `-D *`len`*`

    | 命令行格式 | `--data-file-length=len` |
    | --- | --- |
    | 类型 | 数值 |

    数据文件的最大长度（当重新创建数据文件时，当其“满”时）。

+   `--extend-check`, `-e`

    | 命令行格式 | `--extend-check` |
    | --- | --- |

    进行尝试从数据文件中恢复每一行的修复。通常，这也会找到很多垃圾行。除非你绝望，否则不要使用此选项。

    另请参阅表检查选项下此选项的描述。

    对于输出格式的描述，请参见第 6.6.4.5 节，“使用 myisamchk 获取表信息”。

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    覆盖旧的中间文件（文件名类似`*`tbl_name`*.TMD`）而不是中止。

+   `--keys-used=*`val`*`, `-k *`val`*`

    | 命令行格式 | `--keys-used=val` |
    | --- | --- |
    | 类型 | 数值 |

    对于**myisamchk**，选项值是一个比特值，指示要更新哪些索引。选项值的每个二进制比特对应一个表索引，其中第一个索引是比特 0。选项值为 0 会禁用对所有索引的更新，这可以用于加快插入速度。通过使用**myisamchk -r**可以重新激活停用的索引。

+   `--max-record-length=*`len`*`

    | 命令行格式 | `--max-record-length=len` |
    | --- | --- |
    | 类型 | 数值 |

    如果**myisamchk**无法分配内存来保存大于给定长度的行，则跳过这些行。

+   `--parallel-recover`, `-p`

    | 命令行格式 | `--parallel-recover` |
    | --- | --- |

    注意

    此选项在 MySQL 8.0.28 中已弃用，并在 MySQL 8.0.30 中移除。

    使用与`-r`和`-n`相同的技术，但并行创建所有键，使用不同的线程。*这是测试质量的代码。使用需谨慎！*

+   `--quick`, `-q`

    | 命令行格式 | `--quick` |
    | --- | --- |

    通过仅修改索引文件而不是数据文件来实现更快的修复。您可以两次指定此选项，以强制**myisamchk**在出现重复键的情况下修改原始数据文件。

+   `--recover`, `-r`

    | 命令行格式 | `--recover` |
    | --- | --- |

    进行修复，几乎可以解决除非唯一键不唯一（这是`MyISAM`表极不可能出现的错误）之外的任何问题。如果要恢复表格，这是首选尝试的选项。只有在**myisamchk**报告无法使用`--recover`恢复表格时，才应尝试`--safe-recover`。（在极少数情况下，如果`--recover`失败，数据文件仍然完好无损。）

    如果您有大量内存，应增加`myisam_sort_buffer_size`的值。

+   `--safe-recover`, `-o`

    | 命令行格式 | `--safe-recover` |
    | --- | --- |

    使用一种旧的恢复方法进行修复，该方法按顺序读取所有行并根据找到的行更新所有索引树。这比`--recover`慢一个数量级，但可以处理一些极不可能的情况，而`--recover`无法处理。此恢复方法还比`--recover`使用的磁盘空间少得多。通常，您应该首先使用`--recover`进行修复，然后仅在`--recover`失败时才使用`--safe-recover`。

    如果您有大量内存，应增加`key_buffer_size`的值。

+   `--set-collation=*`name`*`

    | 命令行格式 | `--set-collation=name` |
    | --- | --- |
    | 类型 | 字符串 |

    指定用于对表索引进行排序的校对规则。字符集名称由校对规则名称的第一部分隐含确定。

+   `--sort-recover`, `-n`

    | 命令行格式 | `--sort-recover` |
    | --- | --- |

    强制**myisamchk** 使用排序来解决键的问题，即使临时文件非常大。

+   `--tmpdir=*`dir_name`*`, `-t *`dir_name`*`

    | 命令行格式 | `--tmpdir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    用于存储临时文件的目录路径。如果未设置，**myisamchk** 将使用`TMPDIR`环境变量的值。`--tmpdir` 可以设置为一个目录路径列表，这些目录路径将循环使用以创建临时文件。在 Unix 上目录名称之间的分隔符是冒号（`:`），在 Windows 上是分号（`;`）。

+   `--unpack`, `-u`

    | 命令行格式 | `--unpack` |
    | --- | --- |

    解压使用**myisampack** 打包的表。
