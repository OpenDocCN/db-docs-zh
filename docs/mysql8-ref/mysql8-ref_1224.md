> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-compression-tuning.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-tuning.html)

#### 17.9.1.3 调整 InnoDB 表的压缩

大多数情况下，InnoDB 数据存储和压缩中描述的内部优化确保系统能够很好地运行压缩数据。然而，由于压缩的效率取决于您的数据性质，您可以做出影响压缩表性能的决策：

+   哪些表需要压缩。

+   使用何种压缩页大小。

+   根据运行时性能特征调整缓冲池大小的时机，例如系统花费在压缩和解压数据上的时间量。工作负载更像是数据仓库（主要是查询）还是 OLTP 系统（查询和 DML 混合）。

+   如果系统在压缩表上执行 DML 操作，并且数据分布方式导致运行时昂贵的压缩失败，则可能需要调整其他高级配置选项。

使用本节中的指南来帮助做出这些架构和配置选择。当您准备进行长期测试并将压缩表投入生产时，请参阅第 17.9.1.4 节，“监视 InnoDB 表在运行时的压缩”，以验证这些选择在实际条件下的有效性。

##### 何时使用压缩

一般来说，压缩最适合包含合理数量的字符列并且数据读取频率远远高于写入频率的表。因为没有确切的方法来预测压缩是否对特定情况有益，所以始终要使用特定的工作负载和数据集在代表性配置上进行测试。在决定哪些表需要压缩时，请考虑以下因素。

##### 数据特征和压缩

压缩在减小数据文件大小方面的效率的一个关键因素是数据本身的性质。请记住，压缩通过识别数据块中重复的字节字符串来工作。完全随机化的数据是最糟糕的情况。典型数据通常具有重复值，因此可以有效地压缩。字符字符串通常压缩得很好，无论是在`CHAR`、`VARCHAR`、`TEXT`还是`BLOB`列中定义。另一方面，包含大部分二进制数据（整数或浮点数）或先前压缩的数据（例如<acronym class="acronym">JPEG</acronym>或<acronym class="acronym">PNG</acronym>图像）的表通常可能不会有效地压缩，或者根本不会压缩。

你可以选择是否为每个 InnoDB 表启用压缩。一个表及其所有索引使用相同的（压缩的）页面大小。也许主键（聚簇）索引，其中包含表的所有列的数据，比次要索引更有效地压缩。对于那些存在长行的情况，使用压缩可能导致长列值被存储在“页外”，如 DYNAMIC 行格式中所讨论的那样。这些溢出页面可能会压缩得很好。考虑到这些因素，对于许多应用程序，一些表比其他表更有效地压缩，你可能会发现你的工作负载只有在一部分表被压缩时才能表现最佳。

要确定是否压缩特定表，进行实验。你可以通过使用实现 LZ77 压缩的实用程序（如`gzip`或 WinZip）对未压缩表的.ibd 文件进行压缩的粗略估计。你可以期望 MySQL 压缩表的压缩效率比基于文件的压缩工具要低，因为 MySQL 根据默认的页面大小（16KB）对数据进行分块压缩。除了用户数据，页面格式还包括一些未压缩的内部系统数据。基于文件的压缩工具可以检查更大的数据块，因此可能会在一个巨大的文件中找到比 MySQL 在一个单独页面中找到的更多重复字符串。

另一种测试特定表的压缩的方法是将一些数据从未压缩的表复制到一个类似的、压缩的表（具有完全相同的索引）中，该表位于 file-per-table 表空间中，并查看生成的`.ibd`文件的大小。例如：

```sql
USE test;
SET GLOBAL innodb_file_per_table=1;
SET GLOBAL autocommit=0;

-- Create an uncompressed table with a million or two rows.
CREATE TABLE big_table AS SELECT * FROM information_schema.columns;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
INSERT INTO big_table SELECT * FROM big_table;
COMMIT;
ALTER TABLE big_table ADD id int unsigned NOT NULL PRIMARY KEY auto_increment;

SHOW CREATE TABLE big_table\G

select count(id) from big_table;

-- Check how much space is needed for the uncompressed table.
\! ls -l data/test/big_table.ibd

CREATE TABLE key_block_size_4 LIKE big_table;
ALTER TABLE key_block_size_4 key_block_size=4 row_format=compressed;

INSERT INTO key_block_size_4 SELECT * FROM big_table;
commit;

-- Check how much space is needed for a compressed table
-- with particular compression settings.
\! ls -l data/test/key_block_size_4.ibd
```

这个实验产生了以下数字，当然这些数字可能会有很大的变化，取决于你的表结构和数据：

```sql
-rw-rw----  1 cirrus  staff  310378496 Jan  9 13:44 data/test/big_table.ibd
-rw-rw----  1 cirrus  staff  83886080 Jan  9 15:10 data/test/key_block_size_4.ibd
```

要查看压缩是否对你特定的工作负载有效：

+   对于简单的测试，使用一个没有其他压缩表的 MySQL 实例，并运行查询来访问信息模式`INNODB_CMP`表。

+   对涉及多个压缩表的更复杂测试，运行查询来访问信息模式`INNODB_CMP_PER_INDEX`表。因为`INNODB_CMP_PER_INDEX`表中的统计信息收集起来很昂贵，你必须在查询该表之前启用配置选项`innodb_cmp_per_index_enabled`，并且你可能会将这样的测试限制在开发服务器或非关键复制服务器上。

+   运行一些典型的 SQL 语句来测试你正在测试的压缩表。

+   通过查询`INFORMATION_SCHEMA.INNODB_CMP`或`INFORMATION_SCHEMA.INNODB_CMP_PER_INDEX`来检查成功压缩操作与总体压缩操作的比率，并比较`COMPRESS_OPS`和`COMPRESS_OPS_OK`。

+   如果高比例的压缩操作成功完成，该表可能是压缩的一个良好候选。

+   如果你得到高比例的压缩失败，你可以根据 17.9.1.6 节，“OLTP 工作负载的压缩”中描述的方式调整`innodb_compression_level`、`innodb_compression_failure_threshold_pct` 和 `innodb_compression_pad_pct_max` 选项，并尝试进一步的测试。

##### 数据库压缩与应用程序压缩

决定是在应用程序中压缩数据还是在表中压缩数据；不要同时对相同数据使用两种类型的压缩。当你在应用程序中压缩数据并将结果存储在压缩表中时，额外的空间节省是极不可能的，双重压缩只会浪费 CPU 周期。

##### 在数据库中进行压缩

启用时，MySQL 表压缩是自动的，并适用于所有列和索引值。列仍然可以使用`LIKE`等运算符进行测试，排序操作仍然可以使用索引，即使索引值已经被压缩。由于索引通常占数据库总大小的相当大比例，压缩可能会在存储、I/O 或处理器时间上带来显著的节省。压缩和解压缩操作发生在数据库服务器上，该服务器可能是一个强大的系统，大小适合处理预期的负载。

##### 在应用程序中进行压缩

如果在将数据插入数据库之前在应用程序中压缩数据，您可能会通过对某些列进行压缩而对其他列不进行压缩来节省不易压缩的数据的开销。这种方法在客户端机器上使用 CPU 周期进行压缩和解压缩，而不是在数据库服务器上进行，这可能适用于具有许多客户端的分布式应用程序，或者客户端机器有多余 CPU 周期的情况。

##### 混合方法

当然，也可以结合这些方法。对于某些应用程序，可能适合使用一些压缩表和一些未压缩表。最好是对一些数据进行外部压缩（并将其存储在未压缩表中），并允许 MySQL 在应用程序中压缩（部分）其他表。一如既往，提前设计和实际测试对于做出正确决策是有价值的。

##### 工作负载特征和压缩

除了选择哪些表进行压缩（以及页面大小）之外，工作负载是性能的另一个关键决定因素。如果应用程序主要由读操作组成，而不是更新操作，那么在索引页用完每页“修改日志”空间后，需要重新组织和重新压缩的页面就会减少。如果更新主要更改非索引列或包含`BLOB`或大字符串的列（恰好存储在“页外”），则压缩的开销可能是可以接受的。如果对表的唯一更改是使用单调递增主键进行的`INSERT`，并且几乎没有次要索引，那么几乎没有必要重新组织和重新压缩索引页。由于 MySQL 可以通过修改未压缩数据“原地”在压缩页面上“标记删除”和删除行，对表进行的`DELETE`操作相对高效。

对于某些环境来说，加载数据所需的时间与运行时检索一样重要。特别是在数据仓库环境中，许多表可能是只读或读取频繁的。在这种情况下，除非减少磁盘读取或存储成本的节省显著，否则在加载时间上付出压缩的代价可能是可以接受的，也可能不可接受。

从根本上讲，压缩在 CPU 时间可用于压缩和解压数据时效果最佳。因此，如果您的工作负载受限于 I/O，而不是 CPU，您可能会发现压缩可以提高整体性能。在测试不同压缩配置的应用程序性能时，请在与生产系统计划配置相似的平台上进行测试。

##### 配置特性和压缩

从磁盘读取和写入数据库页是系统性能中最慢的部分。压缩试图通过使用 CPU 时间来压缩和解压数据来减少 I/O，并且在 I/O 相对稀缺的情况下，与处理器周期相比，效果最佳。

当在具有快速、多核 CPU 的多用户环境中运行时，通常尤其如此。当压缩表的一页在内存中时，MySQL 通常会使用额外的内存，通常为 16KB，在缓冲池中存储一页的未压缩副本。自适应 LRU 算法试图在压缩和未压缩页之间平衡内存使用，考虑到工作负载是以 I/O 为限还是以 CPU 为限。然而，与内存高度受限的配置相比，将更多内存专用于缓冲池的配置在使用压缩表时往往运行得更好。

##### 选择压缩页大小

压缩页大小的最佳设置取决于表及其索引包含的数据类型和分布。压缩页大小应始终大于最大记录大小，否则操作可能会失败，如压缩 B-Tree 页中所述。

设置压缩页大小过大会浪费一些空间，但页面不必经常压缩。如果压缩页大小设置过小，则插入或更新可能需要耗时的重新压缩，并且 B 树节点可能需要更频繁地拆分，导致更大的数据文件和不太有效的索引。

通常，您将压缩页大小设置为 8K 或 4K 字节。鉴于 InnoDB 表的最大行大小约为 8K，`KEY_BLOCK_SIZE=8`通常是一个安全选择。
