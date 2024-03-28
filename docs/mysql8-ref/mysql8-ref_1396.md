> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-rules-channel-based-filters.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-rules-channel-based-filters.html)

#### 19.2.5.4 复制通道基于过滤器

本节解释了在存在多个复制通道时如何使用复制过滤器，例如在多源复制拓扑中。在 MySQL 8.0 之前，所有复制过滤器都是全局的，因此过滤器适用于所有复制通道。从 MySQL 8.0 开始，复制过滤器可以是全局的或特定于通道的，使您能够在特定复制通道上配置具有复制过滤器的多源副本。特定于通道的复制过滤器在多源复制拓扑中特别有用，当相同数据库或表存在于多个源上时，并且只需要从一个源复制副本时。

有关设置复制通道的说明，请参见第 19.1.5 节，“MySQL 多源复制”，有关其工作原理的更多信息，请参见第 19.2.2 节，“复制通道”。

重要

多源副本上的每个通道必须从不同源复制。即使您使用复制过滤器在每个通道上选择不同的数据进行复制，也不能从单个副本到单个源设置多个复制通道。这是因为副本的服务器 ID 在复制拓扑中必须是唯一的。源仅通过副本的服务器 ID 来区分副本，而不是通过复制通道的名称，因此它无法识别来自同一副本的不同复制通道。

重要

在配置为组复制的 MySQL 服务器实例上，可以在与组复制无直接关系的复制通道上使用特定于通道的复制过滤器，例如，其中一个组成员还充当来自组外部源的副本。它们不能用于`group_replication_applier`或`group_replication_recovery`通道。在这些通道上进行过滤会使组无法达成一致状态的协议。

重要

对于钻石拓扑结构中的多源复制（其中复制品从两个或更多源复制，这些源源自一个共同的源），当使用基于 GTID 的复制时，请确保多源复制的所有通道上的任何复制过滤器或其他通道配置都是相同的。使用基于 GTID 的复制时，过滤器仅应用于事务数据，而 GTID 不会被过滤掉。这样做是为了使复制品的 GTID 集与源的保持一致，这意味着可以使用 GTID 自动定位而无需每次重新获取被过滤掉的事务。在下游复制品是多源的情况下，并且在钻石拓扑结构中从多个源接收相同事务的情况下，下游复制品现在具有事务的多个版本，结果取决于哪个通道首先应用该事务。尝试的第二个通道使用 GTID 自动跳过跳过事务，因为事务的 GTID 已被第一个通道添加到`gtid_executed`集中。在通道上具有相同过滤器的情况下，没有问题，因为所有事务的版本都包含相同的数据，因此结果是相同的。然而，如果通道上的过滤器不同，数据库可能变得不一致，复制可能会停滞。

##### 复制过滤器和通道概述

当存在多个复制通道时，例如在多源复制拓扑中，复制过滤器应用如下：

+   任何指定的全局复制过滤器都会添加到过滤器类型（`do_db`、`do_ignore_table`等）的全局复制过滤器中。

+   任何特定于通道的复制过滤器都会将过滤器添加到指定通道的指定过滤器类型的复制过滤器中。

+   如果未配置此类型的通道特定复制过滤器，则每个复制通道将全局复制过滤器复制到其特定于通道的复制过滤器中。

+   每个通道使用其特定于通道的复制过滤器来过滤复制流。

创建频道特定复制过滤器的语法扩展了现有的 SQL 语句和命令选项。当未指定复制频道时，全局复制过滤器被配置以确保向后兼容性。`CHANGE REPLICATION FILTER`语句支持`FOR CHANNEL`子句在线配置频道特定过滤器。使用`--replicate-*`命令选项配置过滤器可以指定复制频道，形式为`--replicate-*`filter_type`*=*`channel_name`*:*`filter_details`*`。假设在服务器启动之前存在`channel_1`和`channel_2`频道；在这种情况下，使用命令行选项`--replicate-do-db=db1` `--replicate-do-db=channel_1:db2` `--replicate-do-db=db3` `--replicate-ignore-db=db4` `--replicate-ignore-db=channel_2:db5` `--replicate-wild-do-table=channel_1:db6.t1%` 将导致：

+   *全局复制过滤器*: `do_db=db1,db3`; `ignore_db=db4`

+   *channel_1 上的频道特定过滤器*: `do_db=db2`; `ignore_db=db4`; `wild-do-table=db6.t1%`

+   *channel_2 上的频道特定过滤器*: `do_db=db1,db3`; `ignore_db=db5`

在复制品的`my.cnf`文件中包含时，这些相同规则可以在启动时应用，如下所示：

```sql
replicate-do-db=db1
replicate-do-db=channel_1:db2
replicate-ignore-db=db4
replicate-ignore-db=channel_2:db5
replicate-wild-do-table=db6.channel_1.t1%
```

在这种设置中监视复制过滤器，请使用`replication_applier_global_filters`和`replication_applier_filters`表。

##### 在启动时配置频道特定复制过滤器

与复制过滤器相关的命令选项可以在可选的*`channel`*后跟一个冒号，然后是过滤器规范。第一个冒号被解释为分隔符，后续的冒号被解释为字面冒号。以下命令选项支持使用此格式配置频道特定复制过滤器：

+   `--replicate-do-db=*`channel`*:*`database_id`*`

+   `--replicate-ignore-db=*`channel`*:*`database_id`*`

+   `--replicate-do-table=*`channel`*:*`table_id`*`

+   `--replicate-ignore-table=*`channel`*:*`table_id`*`

+   `--replicate-rewrite-db=*`channel`*:*`db1-db2`*`

+   `--replicate-wild-do-table=*`channel`*:*`table`* *`pattern`*`

+   `--replicate-wild-ignore-table=*`channel`*:*`table`* *`pattern`*`

所有列出的选项都可以在副本的`my.cnf`文件中使用，与大多数其他 MySQL 服务器启动选项一样，省略前两个破折号。参见 Overview of Replication Filters and Channels，以及 Section 6.2.2.2, “Using Option Files”中的简短示例。

如果使用冒号但不为过滤选项指定*`channel`*，例如`--replicate-do-db=:*`database_id`*`，则该选项将为默认复制通道配置复制过滤器。默认复制通道是一旦启动复制就始终存在的复制通道，与手动创建的多源复制通道不同。当既不指定冒号也不指定*`channel`*时，该选项将配置全局复制过滤器，例如`--replicate-do-db=*`database_id`*`配置全局`--replicate-do-db`过滤器。

如果配置了多个`rewrite-db=*`from_name`*->*`to_name`*`选项与相同的*`from_name`*数据库，所有过滤器将被添加在一起（放入`rewrite_do`列表中），第一个生效。

用于`--replicate-wild-*-table`选项的*`pattern`*可以包括标识符中允许的任何字符，以及通配符`%`和`_`。这些与`LIKE`操作符一起使用时的工作方式相同；例如，`tbl%`匹配任何以`tbl`开头的表名，而`tbl_`匹配与`tbl`匹配的表名加一个额外字符。

##### 在线更改通道特定的复制过滤器

除了`--replicate-*`选项外，还可以使用`CHANGE REPLICATION FILTER`语句配置复制过滤器。这样可以避免重新启动服务器，但在进行更改时必须停止复制 SQL 线程。要使此语句将过滤器应用于特定通道，请使用`FOR CHANNEL *`channel`*`子句。例如：

```sql
CHANGE REPLICATION FILTER REPLICATE_DO_DB=(db1) FOR CHANNEL channel_1;
```

当提供`FOR CHANNEL`子句时，语句将作用于指定通道的复制过滤器。如果指定了多种类型的过滤器（`do_db`、`do_ignore_table`、`wild_do_table`等），则只有指定的过滤器类型会被语句替换。例如，在具有多个通道的复制拓扑结构中，例如在多源复制中，如果没有提供`FOR CHANNEL`子句，则语句将作用于全局复制过滤器和所有通道的复制过滤器，使用与`FOR CHANNEL`情况类似的逻辑。更多信息请参见 Section 15.4.2.2, “CHANGE REPLICATION FILTER Statement”。

##### 移除通道特定的复制过滤器

当配置了通道特定的复制过滤器时，可以通过发出空的过滤器类型语句来移除过滤器。例如，要从名为`channel_1`的复制通道中移除所有`REPLICATE_REWRITE_DB`过滤器，请发出：

```sql
CHANGE REPLICATION FILTER REPLICATE_REWRITE_DB=() FOR CHANNEL channel_1;
```

之前使用命令选项或`CHANGE REPLICATION FILTER`配置的任何`REPLICATE_REWRITE_DB`过滤器都会被移除。

`RESET REPLICA ALL` 语句会移除在被语句删除的通道上设置的通道特定的复制过滤器。当被删除的通道重新创建时，任何为副本指定的全局复制过滤器会被复制到它们，而不会应用任何通道特定的复制过滤器。
