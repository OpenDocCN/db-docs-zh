> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html)

#### 6.6.4.4 其他 myisamchk 选项

[**myisamchk**](https://dev.mysql.com/doc/refman/8.0/en/myisamchk.html "6.6.4 myisamchk — MyISAM 表维护实用程序")支持以下选项，用于除表检查和修复之外的操作：

+   [`--analyze`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html#option_myisamchk_analyze)，`-a`

    | 命令行格式 | `--analyze` |
    | --- | --- |

    分析关键值的分布。这通过使连接优化器更好地选择连接表的顺序和应该使用的索引来提高连接性能。要获取关于关键分布的信息，请使用[**myisamchk --description --verbose *`tbl_name`***](https://dev.mysql.com/doc/refman/8.0/en/myisamchk.html "6.6.4 myisamchk — MyISAM 表维护实用程序")命令或`SHOW INDEX FROM *`tbl_name`*`语句。

+   [`--block-search=*`offset`*`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html#option_myisamchk_block-search)，`-b *`offset`*`

    | 命令行格式 | `--block-search=offset` |
    | --- | --- |
    | 类型 | 数字 |

    查找给定偏移处块所属的记录。

+   [`--description`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html#option_myisamchk_description)，`-d`

    | 命令行格式 | `--description` |
    | --- | --- |

    打印有关表的一些描述性信息。指定[`--verbose`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-general-options.html#option_myisamchk_verbose)选项一次或两次会产生额外信息。参见[第 6.6.4.5 节，“使用 myisamchk 获取表信息”](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-table-info.html "6.6.4.5 使用 myisamchk 获取表信息")。

+   [`--set-auto-increment[=*`value`*]`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html#option_myisamchk_set-auto-increment)，`-A[*`value`*]`

    强制`AUTO_INCREMENT`为新记录的编号从给定值开始（如果存在`AUTO_INCREMENT`值大于此值的现有记录，则从更高值开始）。如果未指定*`value`*，则新记录的`AUTO_INCREMENT`编号从表中当前最大值加一开始。

+   [`--sort-index`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html#option_myisamchk_sort-index)，`-S`

    | 命令行格式 | `--sort-index` |
    | --- | --- |

    按高低顺序对索引树块进行排序。这优化了查找并使使用索引的表扫描更快。

+   [`--sort-records=*`N`*`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-other-options.html#option_myisamchk_sort-records)，`-R *`N`*`

    | 命令行格式 | `--sort-records=#` |
    | --- | --- |
    | 类型 | 数字 |

    根据特定索引对记录进行排序。这使您的数据更加本地化，并可能加快使用此索引的基于范围的[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "15.2.13 SELECT 语句")和`ORDER BY`操作的速度。（第一次使用此选项对表进行排序时可能会非常慢。）要确定表的��引编号，请使用[`SHOW INDEX`](https://dev.mysql.com/doc/refman/8.0/en/show-index.html "15.7.7.22 SHOW INDEX 语句")，它以[**myisamchk**](https://dev.mysql.com/doc/refman/8.0/en/myisamchk.html "6.6.4 myisamchk — MyISAM 表维护实用程序")看到的相同顺序显示表的索引。索引从 1 开始编号。

    如果键没有打包（`PACK_KEYS=0`），它们的长度相同，因此当**myisamchk**对记录进行排序和移动时，它只是覆盖索引中的记录偏移量。如果键已经打包（`PACK_KEYS=1`），**myisamchk**必须首先解压键块，然后重新创建索引并再次打包键块。（在这种情况下，重新创建索引比为每个索引更新偏移量更快。）
