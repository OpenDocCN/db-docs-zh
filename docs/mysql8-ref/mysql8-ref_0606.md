# 10.9.5 优化器成本模型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/cost-model.html`](https://dev.mysql.com/doc/refman/8.0/en/cost-model.html)

为生成执行计划，优化器使用基于查询执行过程中各种操作成本的估算的成本模型。优化器具有一组编译默认的“成本常量”可供其在决定执行计划时使用。

优化器还具有成本估算数据库，用于执行计划构建过程中使用。这些估算存储在`mysql`系统数据库的`server_cost`和`engine_cost`表中，并且可以随时进行配置。这些表的目的是使得优化器在尝试生成查询执行计划时能够轻松调整使用的成本估算。

+   成本模型一般操作

+   成本模型数据库

+   对成本模型数据库进行更改

#### 成本模型一般操作

可配置的优化器成本模型工作方式如下：

+   服务器在启动时将成本模型表读入内存，并在运行时使用内存中的值。表中指定的任何非`NULL`成本估算优先于相应的编译默认成本常量。任何`NULL`估算指示优化器使用编译默认值。

+   在运行时，服务器可能会重新读取成本表。当动态加载存储引擎或执行`FLUSH OPTIMIZER_COSTS`语句时会发生这种情况。

+   成本表使服务器管理员可以通过更改表中的条目轻松调整成本估算。通过将条目的成本设置为`NULL`可以轻松恢复默认值。优化器使用内存中的成本值，因此更改表后应跟随`FLUSH OPTIMIZER_COSTS`以生效。

+   当客户端会话开始时，内存中的成本估算值将贯穿整个会话直至结束。特别是，如果服务器重新读取成本表，则任何更改的估算仅适用于随后启动的会话。现有会话不受影响。

+   成本表特定于给定的服务器实例。服务器不会将成本表更改复制到副本。

#### 成本模型数据库

优化器成本模型数据库包括`mysql`系统数据库中的两个表，这些表包含查询执行过程中发生的操作的成本估算信息：

+   `server_cost`：一般服务器操作的优化器成本估算

+   `engine_cost`：特定存储引擎操作的优化器成本估算

`server_cost`表包含以下列：

+   `cost_name`

    用于成本模型的成本估算名称。名称不区分大小写。如果服务器在读取此表时无法识别成本名称，则会将警告写入错误日志。

+   `cost_value`

    成本估算值。如果值为非`NULL`，服务器将其用作成本。否则，它使用默认估算值（编译值）。DBA 可以通过更新此列来更改成本估算。如果服务器在读取此表时发现成本值无效（非正数），则会将警告写入错误日志。

    要覆盖默认成本估算（对于指定`NULL`的条目），请将成本设置为非`NULL`值。要恢复默认值，请将值设置为`NULL`。然后执行`FLUSH OPTIMIZER_COSTS`以告诉服务器重新读取成本表。

+   `last_update`

    最后一次行更新的时间。

+   `comment`

    与成本估算相关联的描述性注释。DBA 可以使用此列提供有关为什么成本估算行存储特定值的信息。

+   `default_value`

    成本估算的默认（编译内）值。此列是只读生成的列，即使关联的成本估算更改，它也保留其值。对于在运行时添加到表中的行，此列的值为`NULL`。

`server_cost`表的主键是`cost_name`列，因此不可能为任何成本估算创建多个条目。

服务器识别`server_cost`表的这些`cost_name`值：

+   `disk_temptable_create_cost`，`disk_temptable_row_cost`

    存储在基于磁盘的存储引擎（`InnoDB`或`MyISAM`）中的内部创建临时表的成本估算。增加这些值会增加使用内部临时表的成本估算，并使优化器更倾向于使用较少的查询计划。有关此类表的信息，请参见 Section 10.4.4, “MySQL 中的内部临时表使用”。

    与相应内存参数（`memory_temptable_create_cost`，`memory_temptable_row_cost`）的默认值相比，这些磁盘参数的默认值较大，反映了处理基于磁盘的表的更高成本。

+   `key_compare_cost`

    比较记录键的成本。增加此值会导致比较许多键的查询计划变得更昂贵。例如，执行`filesort`的查询计划相对于通过使用索引避免排序的查询计划变得更昂贵。

+   `memory_temptable_create_cost`，`memory_temptable_row_cost`

    存储在`MEMORY`存储引擎中的内部创建临时表的成本估算。增加这些值会增加使用内部临时表的成本估算，并使优化器更倾向于使用较少的查询计划。有关这些表的信息，请参见 Section 10.4.4, “MySQL 中的内部临时表使用”。

    与相应磁盘参数的默认值（`disk_temptable_create_cost`，`disk_temptable_row_cost`）相比，这些内存参数的默认值较小，反映了处理基于内存的表的较低成本。

+   `row_evaluate_cost`

    评估记录条件的成本。增加这个值会导致查询计划检查许多行变得比检查较少行的查询计划更昂贵。例如，与读取较少行的范围扫描相比，表扫描变得相对更昂贵。

`engine_cost`表包含以下列：

+   `engine_name`

    适用于此成本估算的存储引擎的名称。名称不区分大小写。如果值为`default`，则适用于所有没有自己命名条目的存储引擎。如果服务器在读取此表时不识别引擎名称，则会将警告写入错误日志。

+   `device_type`

    适用于此成本估算的设备类型。该列用于指定不同存储设备类型（例如硬盘驱动器与固态驱动器）的不同成本估算。目前，此信息未被使用，0 是唯一允许的值。

+   `cost_name`

    与`server_cost`表中相同。

+   `cost_value`

    与`server_cost`表中相同。

+   `last_update`

    与`server_cost`表中相同。

+   `注释`

    与`server_cost`表中相同。

+   `default_value`

    成本估算的默认（编译内）值。此列是一个只读生成的列，即使相关的成本估算发生变化，它也会保留其值。对于在运行时添加到表中的行，此列的值为`NULL`，但有一个例外，即如果该行具有与原始行之一相同的`cost_name`值，则`default_value`列的值与该行相同。

`engine_cost`表的主键是由(`cost_name`, `engine_name`, `device_type`)列组成的元组，因此不可能为这些列中的任何值组合创建多个条目。

服务器识别`engine_cost`表中的这些`cost_name`值：

+   `io_block_read_cost`

    从磁盘读取索引或数据块的成本。增加这个值会导致读取许多磁盘块的查询计划变得比读取较少磁盘块的查询计划更昂贵。例如，与读取较少块的范围扫描相比，表扫描变得相对更昂贵。

+   `memory_block_read_cost`

    类似于`io_block_read_cost`，但表示从内存数据库缓冲区读取索引或数据块的成本。

如果`io_block_read_cost`和`memory_block_read_cost`的值不同，同一查询的两次运行之间可能会导致执行计划的变化。假设内存访问的成本低于磁盘访问的成本。在服务器启动时，数据尚未读入缓冲池之前，可能会得到不同的计划，而在查询运行后，数据已经在内存中。

#### 更改成本模型数据库

对于希望从默认值更改成本模型参数的数据库管理员，尝试将值加倍或减半，并测量效果。

更改`io_block_read_cost`和`memory_block_read_cost`参数最有可能产生有价值的结果。这些参数值使得数据访问方法的成本模型能够考虑从不同来源读取信息的成本；也就是说，从磁盘读取信息的成本与从内存缓冲区中读取信息的成本。例如，其他条件相同的情况下，将`io_block_read_cost`设置为大于`memory_block_read_cost`的值会导致优化器更倾向于选择已经保存在内存中的信息而不是需要从磁盘读取的查询计划。

这个示例展示了如何更改`io_block_read_cost`的默认值：

```sql
UPDATE mysql.engine_cost
  SET cost_value = 2.0
  WHERE cost_name = 'io_block_read_cost';
FLUSH OPTIMIZER_COSTS;
```

这个示例展示了如何仅为`InnoDB`存储引擎更改`io_block_read_cost`的值：

```sql
INSERT INTO mysql.engine_cost
  VALUES ('InnoDB', 0, 'io_block_read_cost', 3.0,
  CURRENT_TIMESTAMP, 'Using a slower disk for InnoDB');
FLUSH OPTIMIZER_COSTS;
```
