# 14.9.6 调整 MySQL 全文搜索

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-fine-tuning.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-fine-tuning.html)

MySQL 的全文搜索功能有很少的用户可调参数。如果你有 MySQL 源码分发，你可以更多地控制全文搜索行为，因为一些更改需要修改源代码。参见 Section 2.8, “从源代码安装 MySQL”。

全文搜索经过精心调整以提高效果。在大多数情况下修改默认行为实际上可能会降低效果。*不要修改 MySQL 源码，除非你知道你在做什么*。

本节描述的大多数全文变量必须在服务器启动时设置。更改它们需要重新启动服务器；不能在服务器运行时修改。

一些变量更改需要重建表中的`FULLTEXT`索引。如何执行此操作将在本节后面给出。

+   配置最小和最大单词长度

+   配置自然语言搜索阈值

+   修改布尔全文搜索运算符

+   字符集修改

+   重建 InnoDB 全文索引

+   优化 InnoDB 全文索引

+   重建 MyISAM 全文索引

#### 配置最小和最大单词长度

要索引的单词的最小和最大长度由 `innodb_ft_min_token_size` 和 `innodb_ft_max_token_size`（对于`InnoDB`搜索索引）以及 `ft_min_word_len` 和 `ft_max_word_len`（对于`MyISAM`）定义。

注意

最小和最大单词长度全文参数不适用于使用 ngram 解析器创建的`FULLTEXT`索引。ngram 标记大小由 `ngram_token_size` 选项定义。

在更改任何这些选项后，重新构建你的`FULLTEXT`索引以使更改生效。例如，要使两个字符的单词可搜索，你可以在选项文件中加入以下行：

```sql
[mysqld]
innodb_ft_min_token_size=2
ft_min_word_len=2
```

然后重新启动服务器并重建您的`FULLTEXT`索引。对于`MyISAM`表，请注意以下有关重建`MyISAM`全文索引的说明中关于**myisamchk**的备注。

#### 配置自然语言搜索阈值

对于`MyISAM`搜索索引，自然语言搜索的 50%阈值取决于所选择的特定加权方案。要禁用它，请查找`storage/myisam/ftdefs.h`中的以下行：

```sql
#define GWS_IN_USE GWS_PROB
```

将该行更改为：

```sql
#define GWS_IN_USE GWS_FREQ
```

然后重新编译 MySQL。在这种情况下，无需重建索引。

注意

通过进行此更改，您*严重*降低了 MySQL 为`MATCH()`函数提供充分相关性值的能力。如果您真的需要搜索这样的常见词，最好使用`IN BOOLEAN MODE`进行搜索，该模式不遵守 50%的阈值。

#### 修改布尔全文搜索运算符

要更改在`MyISAM`表上用于布尔全文搜索的运算符，请设置`ft_boolean_syntax`系统变量。（`InnoDB`没有相应的设置。）此变量可以在服务器运行时更改，但您必须具有足够的权限来设置全局系统变量（请参阅第 7.1.9.1 节，“系统变量权限”）。在这种情况下，不需要重建索引。

#### 字符集修改

对于内置全文解析器，您可以通过以下列表中描述的几种方式更改被视为单词字符的字符集。进行修改后，重新为包含任何`FULLTEXT`索引的每个表重建索引。假设您希望将连字符字符（'-'）视为单词字符。使用以下方法之一：

+   修改 MySQL 源代码：在`storage/innobase/handler/ha_innodb.cc`（对于`InnoDB`）或`storage/myisam/ftdefs.h`（对于`MyISAM`）中查看`true_word_char()`和`misc_word_char()`宏。将`'-'`添加到其中一个宏中，然后重新编译 MySQL。

+   修改字符集文件：这不需要重新编译。`true_word_char()`宏使用“字符类型”表来区分字母和数字与其他字符。 您可以编辑一个字符集 XML 文件中的`<ctype><map>`数组的内容，以指定`'-'`是一个“字母”。然后使用给定的字符集为您的`FULLTEXT`索引。有关`<ctype><map>`数组格式的信息，请参阅第 12.13.1 节，“字符定义数组”。

+   为使用索引列的字符集添加新的排序规则，并修改列以使用该排序规则。有关添加排序规则的一般信息，请参见 Section 12.14, “Adding a Collation to a Character Set”。有关全文索引的特定示例，请参见 Section 14.9.7, “Adding a User-Defined Collation for Full-Text Indexing”。

#### 重建 InnoDB 全文索引

要使更改生效，必须在修改以下任一全文索引变量后重建`FULLTEXT`索引：`innodb_ft_min_token_size`; `innodb_ft_max_token_size`; `innodb_ft_server_stopword_table`; `innodb_ft_user_stopword_table`; `innodb_ft_enable_stopword`; `ngram_token_size`。修改`innodb_ft_min_token_size`、`innodb_ft_max_token_size`或`ngram_token_size`需要重新启动服务器。

要为`InnoDB`表重建`FULLTEXT`索引，请使用`ALTER TABLE`与`DROP INDEX`和`ADD INDEX`选项，以删除并重新创建每个索引。

#### 优化 InnoDB 全文索引

在具有全文索引的表上运行`OPTIMIZE TABLE`会重建全文索引，删除已删除的文档 ID，并在可能的情况下 consololidating 多个相同单词的条目。

要优化全文索引，请启用`innodb_optimize_fulltext_only`并运行`OPTIMIZE TABLE`。

```sql
mysql> set GLOBAL innodb_optimize_fulltext_only=ON;
Query OK, 0 rows affected (0.01 sec)

mysql> OPTIMIZE TABLE opening_lines;
+--------------------+----------+----------+----------+
| Table              | Op       | Msg_type | Msg_text |
+--------------------+----------+----------+----------+
| test.opening_lines | optimize | status   | OK       |
+--------------------+----------+----------+----------+
1 row in set (0.01 sec)
```

为了避免在大表上进行全文索引的长时间重建，您可以使用`innodb_ft_num_word_optimize`选项分阶段执行优化。`innodb_ft_num_word_optimize`选项定义了每次运行`OPTIMIZE TABLE`时优化的单词数。默认设置为 2000，这意味着每次运行`OPTIMIZE TABLE`时会优化 2000 个单词。后续的`OPTIMIZE TABLE`操作将从前一个`OPTIMIZE TABLE`操作结束的地方继续。

#### 重建 MyISAM 全文索引

如果您修改影响索引的全文变量（`ft_min_word_len`、`ft_max_word_len`或`ft_stopword_file`），或者更改停用词文件本身，则在进行更改并重新启动服务器后，必须重建您的`FULLTEXT`索引。

要重建`MyISAM`表的`FULLTEXT`索引，只需进行`QUICK`修复操作即可：

```sql
mysql> REPAIR TABLE *tbl_name* QUICK;
```

或者，如刚才所述使用`ALTER TABLE`。在某些情况下，这可能比修复操作更快。

每个包含任何`FULLTEXT`索引的表必须按照刚才展示的方式进行修复。否则，对该表的查询可能会产生不正确的结果，并且对表的修改会导致服务器将表视为损坏并需要修复。

如果您使用**myisamchk**执行修改`MyISAM`表索引的操作（如修复或分析），`FULLTEXT`索引将使用默认的全文参数值进行重建，包括最小单词长度、最大单词长度和停用词文件，除非您另有规定。这可能导致查询失败。

问题出现在这些参数仅由服务器知道。它们不存储在`MyISAM`索引文件中。如果您已修改了服务器使用的最小或最大单词长度或停用词文件值，为了避免问题，请为**myisamchk**指定与**mysqld**使用的相同的`ft_min_word_len`、`ft_max_word_len`和`ft_stopword_file`值。例如，如果您将最小单词长度设置为 3，您可以像这样使用**myisamchk**修复表：

```sql
myisamchk --recover --ft_min_word_len=3 *tbl_name*.MYI
```

为了确保**myisamchk**和服务器使用相同的全文参数值，将每个值放在选项文件的`[mysqld]`和`[myisamchk]`部分中：

```sql
[mysqld]
ft_min_word_len=3

[myisamchk]
ft_min_word_len=3
```

用于`MyISAM`表索引修改的一种替代方法是使用`REPAIR TABLE`、`ANALYZE TABLE`、`OPTIMIZE TABLE`或`ALTER TABLE`语句。这些语句由服务器执行，服务器知道要使用的正确全文参数值。
