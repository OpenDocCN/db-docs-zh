> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk-table-info.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-table-info.html)

#### 6.6.4.5 使用 myisamchk 获取表信息

要获取 `MyISAM` 表的描述或有关其统计信息，请使用此处显示的命令。稍后将解释这些命令的输出。

+   **myisamchk -d *`tbl_name`***

    在“描述模式”下运行**myisamchk** 以生成表的描述。如果使用外部锁定禁用启动 MySQL 服务器，则在运行时更新的表可能会报告错误。但是，因为**myisamchk** 在描述模式下不会更改表，所以不会破坏数据。

+   **myisamchk -dv *`tbl_name`***

    添加 `-v` 参数以在详细模式下运行**myisamchk**，从而生成有关表的更多信息。再次添加 `-v` 参数会生成更多信息。

+   **myisamchk -eis *`tbl_name`***

    仅显示表中最重要的信息。此操作速度较慢，因为它必须读取整个表。

+   **myisamchk -eiv *`tbl_name`***

    这类似于 `-eis`，但告诉您正在执行什么操作。

*`tbl_name`* 参数可以是 `MyISAM` 表的名称，也可以是其索引文件的名称，如第 6.6.4 节，“myisamchk — MyISAM Table-Maintenance Utility”中所述。可以提供多个 *`tbl_name`* 参数。

假设名为 `person` 的表具有以下结构。（包含 `MAX_ROWS` 表选项，以便在稍后显示的**myisamchk** 示例输出中，某些值较小且更容易适应输出格式。）

```sql
CREATE TABLE person
(
  id         INT NOT NULL AUTO_INCREMENT,
  last_name  VARCHAR(20) NOT NULL,
  first_name VARCHAR(20) NOT NULL,
  birth      DATE,
  death      DATE,
  PRIMARY KEY (id),
  INDEX (last_name, first_name),
  INDEX (birth)
) MAX_ROWS = 1000000 ENGINE=MYISAM;
```

假设表具有以下数据和索引文件大小：

```sql
-rw-rw----  1 mysql  mysql  9347072 Aug 19 11:47 person.MYD
-rw-rw----  1 mysql  mysql  6066176 Aug 19 11:47 person.MYI
```

**myisamchk -dvv** 输出示例：

```sql
MyISAM file:         person
Record format:       Packed
Character set:       utf8mb4_0900_ai_ci (255)
File-version:        1
Creation time:       2017-03-30 21:21:30
Status:              checked,analyzed,optimized keys,sorted index pages
Auto increment key:              1  Last value:                306688
Data records:               306688  Deleted blocks:                 0
Datafile parts:             306688  Deleted data:                   0
Datafile pointer (bytes):        4  Keyfile pointer (bytes):        3
Datafile length:           9347072  Keyfile length:           6066176
Max datafile length:    4294967294  Max keyfile length:   17179868159
Recordlength:                   54

table description:
Key Start Len Index   Type                     Rec/key         Root  Blocksize
1   2     4   unique  long                           1                    1024
2   6     80  multip. varchar prefix                 0                    1024
    87    80          varchar                        0
3   168   3   multip. uint24 NULL                    0                    1024

Field Start Length Nullpos Nullbit Type
1     1     1
2     2     4                      no zeros
3     6     81                     varchar
4     87    81                     varchar
5     168   3      1       1       no zeros
6     171   3      1       2       no zeros
```

**myisamchk** 生成的信息类型解释在此处。 “Keyfile” 指的是索引文件。“Record” 和 “row” 是同义词，同样，“field” 和 “column” 也是。

表描述的初始部分包含这些值：

+   `MyISAM 文件`

    `MyISAM`（索引）文件的名称。

+   `记录格式`

    用于存储表行的格式。前面的示例使用`固定长度`。其他可能的值包括`压缩`和`打包`。（`打包`对应于`SHOW TABLE STATUS`报告的`动态`。）

+   `字符集`

    表的默认字符集。

+   `文件版本`

    `MyISAM`格式的版本。始终为 1。

+   `创建时间`

    数据文件创建时间。

+   `恢复时间`

    上次重建索引/数据文件的时间。

+   `状态`

    表状态标志。可能的值为`崩溃`、`打开`、`更改`、`分析`、`优化键`和`排序索引页`。

+   `自增键`, `最后值`

    与表的`AUTO_INCREMENT`列相关联的键号，以及此列的最近生成的值。如果没有这样的列，则不显示这些字段。

+   `数据记录`

    表中的行数。

+   `已删除块`

    有多少已删除块仍然保留空间。您可以优化表以最小化此空间。请参阅第 9.6.4 节，“MyISAM 表优化”。

+   `数据文件部分`

    对于动态行格式，这表示有多少数据块。对于没有碎片行的优化表，这与`数据记录`相同。

+   `已删除数据`

    有多少字节的未回收删除数据。您可以优化表以最小化此空间。请参阅第 9.6.4 节，“MyISAM 表优化”。

+   `数据文件指针`

    数据文件指针的大小，以字节为单位。通常为 2、3、4 或 5 字节。大多数表使用 2 字节，但目前无法从 MySQL 控制。对于固定表，这是一个行地址。对于动态表，这是一个字节地址。

+   `键文件指针`

    索引文件指针的大小，以字节为单位。通常为 1、2 或 3 字节。大多数表使用 2 字节，但 MySQL 会自动计算。它始终是一个块地址。

+   `最大数据文件长度`

    表数据文件可以变得多长，以字节为单位。

+   `最大键文件长度`

    表索引文件可以变得多长，以字节为单位。

+   `记录长度`

    每行占用多少空间，以字节为单位。

输出的`表描述`部分包括表中所有键的列表。对于每个键，**myisamchk**显示一些低级信息：

+   `键`

    此键的编号。仅对键的第一列显示此值。如果缺少此值，则该行对应于多列键的第二列或更高列。对于示例中显示的表，第二个索引有两行`表描述`。这表示它是一个具有两部分的多部分索引。

+   `开始`

    此索引部分在行中的起始位置。

+   `长度`

    此部分索引的长度。对于压缩数字，这应始终是列的完整长度。对于字符串，它可能比索引列的完整长度短，因为可以索引字符串列的前缀。多部分键的总长度是所有关键部分的`Len`值之和。

+   `Index`

    索引中是否可以存在多个相同的键值。可能的值为`unique`或`multip.`（多个）。

+   `Type`

    此部分索引的数据类型。这是一个`MyISAM`数据类型，可能的值为`packed`、`stripped`或`empty`。

+   `Root`

    根索引块的地址。

+   `Blocksize`

    每个索引块的大小。默认情况下为 1024，但在从源代码构建 MySQL 时，该值可以在编译时更改。

+   `Rec/key`

    这是优化器使用的统计值。它告诉此索引每个值有多少行。唯一索引的值始终为 1。在加载表后（或大幅更改后），可以使用**myisamchk -a**更新此值。如果根本没有更新，将给出默认值 30。

输出的最后部分提供了有关每列的信息：

+   `Field`

    列号。

+   `Start`

    列在表行中的字节位置。

+   `Length`

    列的字节长度。

+   `Nullpos`，`Nullbit`

    对于可以为`NULL`的列，`MyISAM`将`NULL`值存储为一个字节中的标志。根据可为空的列数，可以使用一个或多个字节用于此目的。如果`Nullpos`和`Nullbit`值非空，则指示哪个字节和位包含指示列是否为`NULL`的标志。

    显示用于存储`NULL`标志的位置和字节数的位置和数量在字段 1 的行中显示。这就是为什么`person`表有五列，但有六行`Field`行的原因。

+   `Type`

    数据类型。该值可能包含以下描述符之一：

    +   `constant`

        所有行具有相同的值。

    +   `no endspace`

        不要存储末尾空格。

    +   `no endspace, not_always`

        不要存储末尾空格，也不要对所有值进行末尾空格压缩。

    +   `no endspace, no empty`

        不要存储末尾空格。不要存储空值。

    +   `table-lookup`

        该列已转换为`ENUM`。

    +   `zerofill(*`N`*)`

        值中最重要的*`N`*字节始终为 0 且不存储。

    +   `no zeros`

        不要存储零值。

    +   `always zero`

        零值使用一位存储。

+   `Huff tree`

    与列关联的 Huffman 树的编号。

+   `Bits`

    在 Huffman 树中使用的位数。

如果表已使用**myisampack**进行压缩，则显示`Huff tree`和`Bits`字段。参见第 6.6.6 节，“myisampack — 生成压缩的只读 MyISAM 表”，了解此信息的示例。

**myisamchk -eiv**输出示例：

```sql
Checking MyISAM file: person
Data records:  306688   Deleted blocks:       0
- check file-size
- check record delete-chain
No recordlinks
- check key delete-chain
block_size 1024:
- check index reference
- check data record references index: 1
Key:  1:  Keyblocks used:  98%  Packed:    0%  Max levels:  3
- check data record references index: 2
Key:  2:  Keyblocks used:  99%  Packed:   97%  Max levels:  3
- check data record references index: 3
Key:  3:  Keyblocks used:  98%  Packed:  -14%  Max levels:  3
Total:    Keyblocks used:  98%  Packed:   89%

- check records and index references
**** LOTS OF ROW NUMBERS DELETED ****

Records:            306688  M.recordlength:       25  Packed:            83%
Recordspace used:       97% Empty space:           2% Blocks/Record:   1.00
Record blocks:      306688  Delete blocks:         0
Record data:       7934464  Deleted data:          0
Lost space:         256512  Linkdata:        1156096

User time 43.08, System time 1.68
Maximum resident set size 0, Integral resident set size 0
Non-physical pagefaults 0, Physical pagefaults 0, Swaps 0
Blocks in 0 out 7, Messages in 0 out 0, Signals 0
Voluntary context switches 0, Involuntary context switches 0
Maximum memory usage: 1046926 bytes (1023k)
```

**myisamchk -eiv**输出包括以下信息：

+   `数据记录`

    表中的行数。

+   `已删除块`

    有多少已删除的块仍然保留了空间。您可以优化表以最小化此空间。参见第 9.6.4 节，“MyISAM 表优化”。

+   `键`

    键号。

+   `已使用键块`

    使用的键块的百分比。当表刚刚使用**myisamchk**重新组织时，值非常高（非常接近理论最大值）。

+   `压缩`

    MySQL 尝试压缩具有共同后缀的键值。这仅适用于`CHAR`和`VARCHAR`列上的索引。对于具有相似左侧部分的长索引字符串，这可以显着减少使用的空间。在前面的示例中，第二个键长 40 字节，实现了 97%的空间减少。

+   `最大级别`

    此键的 B 树有多深。具有长键值的大表会获得较高的值。

+   `记录`

    表中有多少行。

+   `M.recordlength`

    平均行长度。对于具有固定长度行的表，这是确切的行长度，因为所有行长度相同。

+   `压缩`

    MySQL 从字符串末尾删除空格。`压缩`值指示通过执行此操作实现的节省百分比。

+   `已使用记录空间`

    数据文件中已使用的百分比。

+   `空间`

    数据文件中未使用的百分比。

+   `块/记录`

    每行的平均块数（即，一个碎片化行由多少个链接组成）。对于固定格式表，这总是 1.0。此值应尽可能接近 1.0。如果值过大，您可以重新组织表。参见第 9.6.4 节，“MyISAM 表优化”。

+   `记录块`

    使用了多少块（链接）。对于固定格式表，这与行数相同。

+   `删除块`

    已删除（未使用）的块（链接）数。

+   `记录数据`

    数据文件中已使用的字节数。

+   `已删除数据`

    数据文件中已删除（未使用）的字节数。

+   `丢失空间`

    如果将行更新为较短长度，则会丢失一些空间。这是所有此类损���的总和，以字节为单位。

+   `链接数据`

    当使用动态表格式时，行片段通过指针（每个 4 到 7 个字节）连接。`Linkdata`是所有这些指针使用的存储量的总和。
