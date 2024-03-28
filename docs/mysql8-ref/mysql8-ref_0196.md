# 6.6.3 myisam_ftdump — 显示全文索引信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-ftdump.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-ftdump.html)

**myisam_ftdump**显示`MyISAM`表中`FULLTEXT`索引的信息。它直接读取`MyISAM`索引文件，因此必须在表所在的服务器主机上运行。在使用**myisam_ftdump**之前，请确保首先发出`FLUSH TABLES`语句（如果服务器正在运行）。

**myisam_ftdump**扫描并转储整个索引，这并不特别快。另一方面，单词的分布变化不频繁，因此不需要经常运行。

像这样调用**myisam_ftdump**：

```sql
myisam_ftdump [*options*] *tbl_name* *index_num*
```

*`tbl_name`*参数应该是一个`MyISAM`表的名称。您还可以通过命名其索引文件（具有`.MYI`后缀的文件）来指定表。如果您不在表文件所在的目录中调用**myisam_ftdump**，则表或索引文件名必须在表的数据库目录的路径名之前。索引编号从 0 开始。

例如：假设`test`数据库包含一个名为`mytexttable`的表，其定义如下：

```sql
CREATE TABLE mytexttable
(
  id   INT NOT NULL,
  txt  TEXT NOT NULL,
  PRIMARY KEY (id),
  FULLTEXT (txt)
) ENGINE=MyISAM;
```

`id`上的索引是索引 0，`txt`上的`FULLTEXT`索引是索引 1。如果您的工作目录是`test`数据库目录，请按以下方式调用**myisam_ftdump**：

```sql
myisam_ftdump mytexttable 1
```

如果`test`数据库目录的路径名为`/usr/local/mysql/data/test`，您也可以使用该路径名指定表名参数。如果您不在数据库目录中调用**myisam_ftdump**，这将非常有用：

```sql
myisam_ftdump /usr/local/mysql/data/test/mytexttable 1
```

您可以在类 Unix 系统上像这样使用**myisam_ftdump**按出现频率生成索引条目列表：

```sql
myisam_ftdump -c mytexttable 1 | sort -r
```

在 Windows 上使用：

```sql
myisam_ftdump -c mytexttable 1 | sort /R
```

**myisam_ftdump**支持以下选项：

+   `--help`, `-h` `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。

+   `--count`, `-c`

    | 命令行格式 | `--count` |
    | --- | --- |

    计算每个单词的统计信息（计数和全局权重）。

+   `--dump`, `-d`

    | 命令行格式 | `--dump` |
    | --- | --- |

    转储索引，包括数据偏移和单词权重。

+   `--length`, `-l`

    | 命令行格式 | `--length` |
    | --- | --- |

    报告长度分布。

+   `--stats`, `-s`

    | 命令行格式 | `--stats` |
    | --- | --- |

    报告全局索引统计。如果没有指定其他操作，则这是默认操作。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。打印程序执行的更多输出。
