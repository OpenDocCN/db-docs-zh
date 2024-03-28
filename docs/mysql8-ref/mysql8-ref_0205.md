# 6.6.6 myisampack — 生成压缩的只读 MyISAM 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisampack.html`](https://dev.mysql.com/doc/refman/8.0/en/myisampack.html)

**myisampack** 实用程序用于压缩`MyISAM`表。**myisampack** 通过分别压缩表中的每一列来工作。通常，**myisampack** 将数据文件压缩 40% 到 70%。

当稍后使用表时，服务器会读取解压缩列所需的信息到内存中。这样在访问单个行时会有更好的性能，因为你只需要解压缩一个行。

MySQL 在可能的情况下使用`mmap()`对压缩表执行内存映射。如果`mmap()`无法工作，MySQL 将退回到正常的读/写文件操作。

请注意以下事项：

+   如果使用外部锁定禁用启动了**mysqld**服务器，如果在打包过程中服务器可能更新表，那么调用**myisampack**不是一个好主意。最安全的方法是在服务器停止时压缩表。

+   对表进行打包后，它变为只读。这通常是有意为之的（比如在 CD 上访问打包的表时）。

+   **myisampack** 不支持分区表。

像这样调用**myisampack**：

```sql
myisampack [*options*] *file_name* ...
```

每个文件名参数应该是一个索引（`.MYI`）文件的名称。如果你不在数据库目录中，你应该指定文件的路径名。可以省略`.MYI`扩展名。

在使用**myisampack**压缩表后，请使用**myisamchk -rq**重建其索引。6.6.4 “myisamchk — MyISAM 表维护实用程序”。

**myisampack** 支持以下选项。它还读取选项文件并支持处理它们的选项，详见 6.2.2.3 “影响选项文件处理的命令行选项”。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |

    显示帮助消息并退出。

+   `--backup`, `-b`

    | 命令行格式 | `--backup` |
    | --- | --- |

    使用名称`*`tbl_name`*.OLD`备份每个表的数据文件。

+   `--character-sets-dir=*`dir_name`*`

    | 命令行格式 | `--character-sets-dir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    安装字符集的目录。参见第 12.15 节，“字符集配置”。

+   [`--debug[=*`debug_options`*]`](myisampack.html#option_myisampack_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug[=debug_options]` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `d:t:o` |

    写入调试日志。典型的*`debug_options`*字符串是`d:t:o,*`file_name`*`。默认为`d:t:o`。

    只有在使用`WITH_DEBUG`构建 MySQL 时才可用此选项。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--force`, `-f`

    | 命令行格式 | `--force` |
    | --- | --- |

    即使生成的打包表比原始表大，或者如果之前调用**myisampack**的中间文件存在，也要生成打包表。(**myisampack**在压缩表时在数据库目录中创建一个名为`*`tbl_name`*.TMD`的中间文件。如果终止**myisampack**，可能不会删除`.TMD`文件。)通常，如果发现`*`tbl_name`*.TMD`存在，**myisampack**会因此而出错。使用`--force`，**myisampack**仍会打包表。

+   `--join=*`big_tbl_name`*`, `-j *`big_tbl_name`*`

    | 命令行格式 | `--join=big_tbl_name` |
    | --- | --- |
    | 类型 | 字符串 |

    将命令行中命名的所有表连接成一个单独的打包表*`big_tbl_name`*。所有要合并的表*必须*具有相同的结构（相同的列名和类型，相同的索引等）。

    在连接操作之前，*`big_tbl_name`*不能存在。要合并到*`big_tbl_name`*的命令行中命名的所有源表必须存在。源表用于连接操作，但不会被修改。

+   `--silent`, `-s`

    | 命令行格式 | `--silent` |
    | --- | --- |

    静默模式。仅在发生错误时输出。

+   `--test`, `-t`

    | 命令行格式 | `--test` |
    | --- | --- |

    不实际打包表，只是测试打包。

+   `--tmpdir=*`dir_name`*`, `-T *`dir_name`*`

    | 命令行格式 | `--tmpdir=dir_name` |
    | --- | --- |
    | 类型 | 目录名称 |

    使用命名目录作为**myisampack**创建临时文件的位置。

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |

    详细模式。写入有关打包操作进展及其结果的信息。

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |

    显示版本信息并退出。

+   `--wait`, `-w`

    | 命令行格式 | `--wait` |
    | --- | --- |

    如果表正在使用，则等待并重试。如果**mysqld**服务器是在禁用外部锁定的情况下调用的，则在打包过程中服务器可能更新表时，调用**myisampack**不是一个好主意。

以下命令序列展示了一个典型的表压缩会话：

```sql
$> ls -l station.*
-rw-rw-r--   1 jones    my         994128 Apr 17 19:00 station.MYD
-rw-rw-r--   1 jones    my          53248 Apr 17 19:00 station.MYI

$> myisamchk -dvv station

MyISAM file:     station
Isam-version:  2
Creation time: 1996-03-13 10:08:58
Recover time:  1997-02-02  3:06:43
Data records:              1192  Deleted blocks:              0
Datafile parts:            1192  Deleted data:                0
Datafile pointer (bytes):     2  Keyfile pointer (bytes):     2
Max datafile length:   54657023  Max keyfile length:   33554431
Recordlength:               834
Record format: Fixed length

table description:
Key Start Len Index   Type                 Root  Blocksize    Rec/key
1   2     4   unique  unsigned long        1024       1024          1
2   32    30  multip. text                10240       1024          1

Field Start Length Type
1     1     1
2     2     4
3     6     4
4     10    1
5     11    20
6     31    1
7     32    30
8     62    35
9     97    35
10    132   35
11    167   4
12    171   16
13    187   35
14    222   4
15    226   16
16    242   20
17    262   20
18    282   20
19    302   30
20    332   4
21    336   4
22    340   1
23    341   8
24    349   8
25    357   8
26    365   2
27    367   2
28    369   4
29    373   4
30    377   1
31    378   2
32    380   8
33    388   4
34    392   4
35    396   4
36    400   4
37    404   1
38    405   4
39    409   4
40    413   4
41    417   4
42    421   4
43    425   4
44    429   20
45    449   30
46    479   1
47    480   1
48    481   79
49    560   79
50    639   79
51    718   79
52    797   8
53    805   1
54    806   1
55    807   20
56    827   4
57    831   4

$> myisampack station.MYI
Compressing station.MYI: (1192 records)
- Calculating statistics

normal:     20  empty-space:   16  empty-zero:     12  empty-fill:  11
pre-space:   0  end-space:     12  table-lookups:   5  zero:         7
Original trees:  57  After join: 17
- Compressing file
87.14%
Remember to run myisamchk -rq on compressed tables

$> myisamchk -rq station
- check record delete-chain
- recovering (with sort) MyISAM-table 'station'
Data records: 1192
- Fixing index 1
- Fixing index 2

$> mysqladmin -uroot flush-tables

$> ls -l station.*
-rw-rw-r--   1 jones    my         127874 Apr 17 19:00 station.MYD
-rw-rw-r--   1 jones    my          55296 Apr 17 19:04 station.MYI

$> myisamchk -dvv station

MyISAM file:     station
Isam-version:  2
Creation time: 1996-03-13 10:08:58
Recover time:  1997-04-17 19:04:26
Data records:               1192  Deleted blocks:              0
Datafile parts:             1192  Deleted data:                0
Datafile pointer (bytes):      3  Keyfile pointer (bytes):     1
Max datafile length:    16777215  Max keyfile length:     131071
Recordlength:                834
Record format: Compressed

table description:
Key Start Len Index   Type                 Root  Blocksize    Rec/key
1   2     4   unique  unsigned long       10240       1024          1
2   32    30  multip. text                54272       1024          1

Field Start Length Type                         Huff tree  Bits
1     1     1      constant                             1     0
2     2     4      zerofill(1)                          2     9
3     6     4      no zeros, zerofill(1)                2     9
4     10    1                                           3     9
5     11    20     table-lookup                         4     0
6     31    1                                           3     9
7     32    30     no endspace, not_always              5     9
8     62    35     no endspace, not_always, no empty    6     9
9     97    35     no empty                             7     9
10    132   35     no endspace, not_always, no empty    6     9
11    167   4      zerofill(1)                          2     9
12    171   16     no endspace, not_always, no empty    5     9
13    187   35     no endspace, not_always, no empty    6     9
14    222   4      zerofill(1)                          2     9
15    226   16     no endspace, not_always, no empty    5     9
16    242   20     no endspace, not_always              8     9
17    262   20     no endspace, no empty                8     9
18    282   20     no endspace, no empty                5     9
19    302   30     no endspace, no empty                6     9
20    332   4      always zero                          2     9
21    336   4      always zero                          2     9
22    340   1                                           3     9
23    341   8      table-lookup                         9     0
24    349   8      table-lookup                        10     0
25    357   8      always zero                          2     9
26    365   2                                           2     9
27    367   2      no zeros, zerofill(1)                2     9
28    369   4      no zeros, zerofill(1)                2     9
29    373   4      table-lookup                        11     0
30    377   1                                           3     9
31    378   2      no zeros, zerofill(1)                2     9
32    380   8      no zeros                             2     9
33    388   4      always zero                          2     9
34    392   4      table-lookup                        12     0
35    396   4      no zeros, zerofill(1)               13     9
36    400   4      no zeros, zerofill(1)                2     9
37    404   1                                           2     9
38    405   4      no zeros                             2     9
39    409   4      always zero                          2     9
40    413   4      no zeros                             2     9
41    417   4      always zero                          2     9
42    421   4      no zeros                             2     9
43    425   4      always zero                          2     9
44    429   20     no empty                             3     9
45    449   30     no empty                             3     9
46    479   1                                          14     4
47    480   1                                          14     4
48    481   79     no endspace, no empty               15     9
49    560   79     no empty                             2     9
50    639   79     no empty                             2     9
51    718   79     no endspace                         16     9
52    797   8      no empty                             2     9
53    805   1                                          17     1
54    806   1                                           3     9
55    807   20     no empty                             3     9
56    827   4      no zeros, zerofill(2)                2     9
57    831   4      no zeros, zerofill(1)                2     9
```

**myisampack**显示以下类型的信息：

+   `正常`

    未使用额外打包的列数。

+   `空格空间`

    只包含空格值的列数。这些占用一个位。

+   `空零`

    只包含二进制零值的列数。这些占用一个位。

+   `空填充`

    不占据其类型的完整字节范围的整数列数。这些将被更改为较小的类型。例如，`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")列（八个字节）可以存储为`TINYINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")列（一个字节），如果所有值都在`-128`到`127`的范围内。

+   `前置空格`

    存储有前导空格的十进制列数。在这种情况下，每个值都包含前导空格的计数。

+   `结束空间`

    具有大量尾随空格的列数。在这种情况下，每个值都包含尾随空格的计数。

+   `表查找`

    该列只有少量不同的值，这些值在 Huffman 压缩之前被转换为`ENUM`。

+   `零`

    所有值都为零的列数。

+   `原始树`

    初始 Huffman 树的数量。

+   `连接后`

    在连接树以节省一些标题空间后剩余的不同 Huffman 树的数量。

表格压缩后，由**myisamchk -dvv**显示的`Field`行包括有关每列的附加信息：

+   `类型`

    数据类型。值可能包含以下描述符之一：

    +   `常量`

        所有行具有相同的值。

    +   `无末尾空格`

        不要存储末尾空格。

    +   `无末尾空格，非始终`

        不要存储末尾空格，也不要对所有值进行末尾空格压缩。

    +   `无末尾空格，无空值`

        不要存储末尾空格。不要存储空值。

    +   `表查找`

        该列已转换为`ENUM`。

    +   `零填充(*`N`*)`

        值中最重要的*`N`*字节始终为 0 且不存储。

    +   `无零值`

        不要存储零值。

    +   `始终为零`

        零值使用一位存储。

+   `Huff 树`

    与列相关的哈夫曼树的编号。

+   `位`

    哈夫曼树中使用的位数。

运行**myisampack**后，使用**myisamchk**重新创建任何索引。此时，您还可以对索引块进行排序，并创建 MySQL 优化器需要的统计信息，以使其更高效地工作：

```sql
myisamchk -rq --sort-index --analyze *tbl_name*.MYI
```

将打包的表安装到 MySQL 数据库目录后，应执行**mysqladmin flush-tables**以强制**mysqld**开始使用新表。

要解压缩打包的表，请使用`--unpack`选项到**myisamchk**。
