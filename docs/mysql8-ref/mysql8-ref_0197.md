# 6.6.4 myisamchk — MyISAM 表维护实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk.html)

6.6.4.1 myisamchk 常规选项

6.6.4.2 myisamchk 检查选项

6.6.4.3 myisamchk 修复选项

6.6.4.4 其他 myisamchk 选项

6.6.4.5 使用 myisamchk 获取表信息

6.6.4.6 myisamchk 内存使用

**myisamchk**实用程序可以获取关于数据库表的信息，或检查、修复或优化它们。**myisamchk**适用于`MyISAM`表（用于存储数据和索引的`.MYD`和`.MYI`文件的表）。

您还可以使用`CHECK TABLE`和`REPAIR TABLE`语句来检查和修复`MyISAM`表。请参阅第 15.7.3.2 节，“CHECK TABLE 语句”和第 15.7.3.5 节，“REPAIR TABLE 语句”。

使用**myisamchk**来处理分区表是不被支持的。

注意

在执行表修复操作之前最好备份表格；在某些情况下，该操作可能导致数据丢失。可能的原因包括但不限于文件系统错误。

像这样调用**myisamchk**：

```sql
myisamchk [*options*] *tbl_name* ...
```

*`options`*指定了你希望**myisamchk**执行的操作。它们在以下章节中描述。你也可以通过调用**myisamchk --help**来获取选项列表。

没有选项时，**myisamchk**仅作为默认操作检查您的表。要获取更多信息或告诉**myisamchk**采取纠正措施，请按照以下讨论中描述的选项。

*`tbl_name`*是你想要检查或修复的数据库表。如果你在数据库目录之外的地方运行**myisamchk**，你必须指定数据库目录的路径，因为**myisamchk**不知道数据库的位置。事实上，**myisamchk**实际上并不关心你正在处理的文件是否位于数据库目录中。你可以将对应于数据库表的文件复制到其他位置，并在那里执行恢复操作。

如果需要，你可以在**myisamchk**命令行上命名多个表。你还可以通过命名其索引文件（具有`.MYI`后缀的文件）来指定一个表。这使你可以通过使用模式`*.MYI`来指定目录中的所有表。例如，如果你在一个数据库目录中，可以像这样检查该目录中的所有`MyISAM`表：

```sql
myisamchk *.MYI
```

如果你不在数据库目录中，可以通过指定目录路径来检查所有表：

```sql
myisamchk */path/to/database_dir/**.MYI
```

甚至可以通过在 MySQL 数据目录路径中指定通配符来检查所有数据库中的所有表：

```sql
myisamchk */path/to/datadir/*/**.MYI
```

快速检查所有`MyISAM`表的推荐方法是：

```sql
myisamchk --silent --fast */path/to/datadir/*/**.MYI
```

如果你想检查所有损坏的`MyISAM`表并修复它们，可以使用以下命令：

```sql
myisamchk --silent --force --fast --update-state \
          --key_buffer_size=64M --myisam_sort_buffer_size=64M \
          --read_buffer_size=1M --write_buffer_size=1M \
          */path/to/datadir/*/**.MYI
```

这个命令假定你有超过 64MB 的空闲内存。有关使用**myisamchk**进行内存分配的更多信息，请参阅第 6.6.4.6 节，“myisamchk 内存使用”。

有关使用**myisamchk**的更多信息，请参阅第 9.6 节，“MyISAM 表维护和崩溃恢复”。

重要

*在运行**myisamchk**时，你必须确保没有其他程序正在使用这些表*。最有效的方法是在运行**myisamchk**时关闭 MySQL 服务器，或者锁定所有**myisamchk**正在使用的表。

否则，当你运行**myisamchk**时，可能会显示以下错误消息：

```sql
warning: clients are using or haven't closed the table properly
```

这意味着您正在尝试检查已被另一个程序（如**mysqld**服务器）更新但���未关闭文件或因未正确关闭文件而死亡的表，这有时会导致一个或多个`MyISAM`表的损坏。

如果**mysqld**正在运行，您必须通过使用`FLUSH TABLES`强制它刷新仍在内存中缓冲的任何表修改。然后在运行**myisamchk**时，确保没有人在使用这些表。

然而，避免这个问题的最简单方法是使用`CHECK TABLE`而不是**myisamchk**来检查表。请参见 Section 15.7.3.2, “CHECK TABLE Statement”。

**myisamchk** 支持以下选项，可以在命令行或选项文件的`[myisamchk]`组中指定。有关 MySQL 程序使用的选项文件的信息，请参见 Section 6.2.2.2, “Using Option Files”。

**表 6.20 myisamchk 选项**

| 选项名称 | 描述 |
| --- | --- |
| --analyze | 分析关键值的分布情况 |
| --backup | 将 .MYD 文件备份为 file_name-time.BAK |
| --block-search | 查找给定偏移量处块所属的记录 |
| --character-sets-dir | 存放字符集的目录 |
| --check | 检查表中的错误 |
| --check-only-changed | 仅检查自上次检查以来发生变化的表 |
| --correct-checksum | 为表正确校验和信息 |
| --data-file-length | 数据文件的最大长度（当重新创建数据文件时文件已满时） |
| --debug | 写入调试日志 |
| --decode_bits | Decode_bits |
| --defaults-extra-file | 除了通常的选项文件外，还读取指定的选项文件 |
| --defaults-file | 仅读取指定的选项文件 |
| --defaults-group-suffix | 选项组后缀值 |
| --description | 打印关于表的一些描述性信息 |
| --extend-check | 进行非常彻底的表检查或修复，尝试从数据文件中恢复每一行 |
| --fast | 仅检查未正确关闭的表 |
| --force | 如果 myisamchk 在表中发现任何错误，则自动执行修复操作 |
| --force | 覆盖旧临时文件。与-r 或-o 选项一起使用 |
| --ft_max_word_len | FULLTEXT 索引的最大单词长度 |
| --ft_min_word_len | FULLTEXT 索引的最小单词长度 |
| --ft_stopword_file | 使用此文件中的停用词而不是内置列表 |
| --HELP | 显示帮助信息并退出 |
| --help | 显示帮助信息并退出 |
| --information | 打印有关已检查表的信息统计 |
| --key_buffer_size | 用于 MyISAM 表的索引块的缓冲区大小 |
| --keys-used | 指示要更新哪些索引的位值 |
| --max-record-length | 如果 myisamchk 无法分配内存来保存它们，则跳过大于给定长度的行 |
| --medium-check | 执行比--extend-check 操作更快的检查 |
| --myisam_block_size | 用于 MyISAM 索引页的块大小 |
| --myisam_sort_buffer_size | 在执行 REPAIR 或使用 CREATE INDEX 或 ALTER TABLE 创建索引时，用于排序索引的缓冲区 |
| --no-defaults | 不读取任何选项文件 |
| --parallel-recover | 使用与-r 和-n 相同的技术，但使用不同线程并行创建所有键（测试版） |
| --print-defaults | 打印默认选项 |
| --quick | 通过不修改数据文件来实现更快的修复 |
| --read_buffer_size | 每个执行顺序扫描的线程为其扫描的每个表分配此大小的缓冲区 |
| --read-only | 不要将表标记为已检查 |
| --recover | 进行修复，可以解决几乎所有问题，除了不唯一的唯一键 |
| --safe-recover | 使用旧的恢复方法进行修复，按顺序读取所有行并根据找到的行更新所有索引树 |
| --set-auto-increment | 强制新记录的 AUTO_INCREMENT 编号从给定值开始 |
| --set-collation | 指定用于排序表索引的排序规则 |
| --silent | 静默模式 |
| --sort_buffer_size | 在执行 REPAIR 或使用 CREATE INDEX 或 ALTER TABLE 创建索引时，用于排序索引的缓冲区 |
| --sort-index | 按高低顺序对索引树块进行排序 |
| --sort_key_blocks | sort_key_blocks |
| --sort-records | 根据特定索引对记录进行排序 |
| --sort-recover | 强制 myisamchk 使用排序来解决键的问题，即使临时文件非常大 |
| --stats_method | 指定 MyISAM 索引统计收集代码如何处理 NULL 值 |
| --tmpdir | 用于存储临时文件的目录 |
| --unpack | 解压使用 myisampack 打包的表 |
| --update-state | 在.MYI 文件中存储信息，指示表何时被检查以及表是否崩溃 |
| --verbose | 详细模式 |
| --version | 显示版本信息并退出 |
| --wait | 等待被锁定的表解锁，而不是终止 |
| --write_buffer_size | 写缓冲区大小 |
| 选项名称 | 描述 |
