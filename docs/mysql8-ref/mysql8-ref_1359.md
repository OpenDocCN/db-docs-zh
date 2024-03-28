> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-provision-replica.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-provision-replica.html)

#### 19.1.5.2 为基于 GTID 的复制提供多源副本

如果多源复制拓扑中的源具有现有数据，可以在开始复制之前使用相关数据为副本提供数据以节省时间。在多源复制拓扑中，无法使用克隆或复制数据目录来为副本提供来自所有源的数据，您可能还想要仅从每个源复制特定数据库。因此，为此类副本提供数据的最佳策略是使用**mysqldump**在每个源上创建适当的转储文件，然后使用**mysql**客户端在副本上导入转储文件。

如果您正在使用基于 GTID 的复制，请注意`SET @@GLOBAL.gtid_purged`语句，该语句由**mysqldump**放置在转储输出中。此语句将源上执行的事务的 GTID 传输到副本，副本需要这些信息。然而，对于比从一个源为一个新的空副本提供更复杂的情况，您需要检查该语句在副本使用的 MySQL 版本中的影响，并相应处理该语句。以下指导总结了适当的操作，但更多详情请参阅**mysqldump**文档。

由**mysqldump**编写的`SET @@GLOBAL.gtid_purged`语句在 MySQL 8.0 的版本中与 MySQL 5.6 和 5.7 中的行为不同。在 MySQL 5.6 和 5.7 中，该语句替换了副本上的`gtid_purged`的值，并且在这些版本中，只有在副本的具有 GTID 的事务记录（`gtid_executed`集）为空时才能更改该值。在多源复制拓扑中，因此在重放转储文件之前必须从转储输出中删除`SET @@GLOBAL.gtid_purged`语句，因为不能应用包含此语句的第二个或后续转储文件。还要注意，在 MySQL 5.6 和 5.7 中，这个限制意味着必须在具有空的`gtid_executed`集的副本上以单个操作应用所有源的转储文件。您可以通过在副本上发出`RESET MASTER`来清除副本的 GTID 执行历史，但如果在副本上有其他需要的带有 GTID 的事务，请选择从 Section 19.1.3.5, “Using GTIDs for Failover and Scaleout”中描述的方法中选择一种替代方法进行配置。

从 MySQL 8.0 开始，`SET @@GLOBAL.gtid_purged`语句将从转储文件中的 GTID 集添加到副本上现有的`gtid_purged`集中。因此，在副本上重放转储文件时，该语句可能会留在转储输出中，并且可以在不同时间重放转储文件。然而，重要的是要注意，**mysqldump**为`SET @@GLOBAL.gtid_purged`语句包含的值包括源上`gtid_executed`集中所有事务的 GTID，即使这些事务更改了数据库的被抑制部分，或者服务器上未包含在部分转储中的其他数据库。如果在副本上重放包含任何相同 GTID 的第二个或后续转储文件（例如，来自相同源的另一个部分转储，或者来自具有重叠事务的另一个源的转储），第二个转储文件中的任何`SET @@GLOBAL.gtid_purged`语句将失败，因此必须从转储输出中删除。

对于来自 MySQL 8.0.17 的源，作为删除`SET @@GLOBAL.gtid_purged`语句的替代方案，您可以将**mysqldump**的`--set-gtid-purged`选项设置为`COMMENTED`，以包含该语句但注释掉，这样在加载转储文件时不会执行该语句。如果您正在使用来自同一源的两个部分转储文件为副本提供数据，并且第二个转储文件中的 GTID 集合与第一个相同（因此在转储之间源上没有执行新事务），则可以在输出第二个转储文件时将**mysqldump**的`--set-gtid-purged`选项设置为`OFF`，以省略该语句。

在以下配置示例中，我们假设`SET @@GLOBAL.gtid_purged`语句不能保留在转储输出中，并且必须从���件中删除并手动处理。我们还假设在开始配置之前，在副本上没有带有 GTID 的想要的事务。

1.  要为名为`source1`上的名为`db1`的数据库和名为`source2`上的名为`db2`的数据库创建转储文件，请按以下方式为`source1`运行**mysqldump**：

    ```sql
    mysqldump -u<*user*> -p<*password*> --single-transaction --triggers --routines --set-gtid-purged=ON --databases db1 > dumpM1.sql
    ```

    然后按以下方式为`source2`运行**mysqldump**：

    ```sql
    mysqldump -u<*user*> -p<*password*> --single-transaction --triggers --routines --set-gtid-purged=ON --databases db2 > dumpM2.sql
    ```

1.  记录`gtid_purged`值，该值是**mysqldump**添加到每个转储文件中的。例如，对于在 MySQL 5.6 或 5.7 上创建的转储文件，可以像这样提取该值：

    ```sql
    cat dumpM1.sql | grep GTID_PURGED | cut -f2 -d'=' | cut -f2 -d$'\''
    cat dumpM2.sql | grep GTID_PURGED | cut -f2 -d'=' | cut -f2 -d$'\''
    ```

    从 MySQL 8.0 开始，格式已更改，您可以像这样提取该值：

    ```sql
    cat dumpM1.sql | grep GTID_PURGED | perl -p0 -e 's#/\*.*?\*/##sg' | cut -f2 -d'=' | cut -f2 -d$'\''
    cat dumpM2.sql | grep GTID_PURGED | perl -p0 -e 's#/\*.*?\*/##sg' | cut -f2 -d'=' | cut -f2 -d$'\''
    ```

    每种情况下的结果应该是一个 GTID 集合，例如：

    ```sql
    source1:   2174B383-5441-11E8-B90A-C80AA9429562:1-1029
    source2:   224DA167-0C0C-11E8-8442-00059A3C7B00:1-2695
    ```

1.  从每个转储文件中删除包含`SET @@GLOBAL.gtid_purged`语句的行。例如：

    ```sql
    sed '/GTID_PURGED/d' dumpM1.sql > dumpM1_nopurge.sql
    sed '/GTID_PURGED/d' dumpM2.sql > dumpM2_nopurge.sql
    ```

1.  使用**mysql**客户端将每个编辑过的转储文件导入到副本中。例如：

    ```sql
    mysql -u<*user*> -p<*password*> < dumpM1_nopurge.sql
    mysql -u<*user*> -p<*password*> < dumpM2_nopurge.sql
    ```

1.  在副本上，发出`RESET MASTER`以清除 GTID 执行历史记录（假设，如上所述，所有转储文件都已导入，并且在副本上没有带有 GTID 的想要的事务）。然后发出`SET @@GLOBAL.gtid_purged`语句，将`gtid_purged`值设置为所有转储文件中所有 GTID 集合的并集，就像在第 2 步中记录的那样。例如：

    ```sql
    mysql> RESET MASTER;
    mysql> SET @@GLOBAL.gtid_purged = "2174B383-5441-11E8-B90A-C80AA9429562:1-1029, 224DA167-0C0C-11E8-8442-00059A3C7B00:1-2695";
    ```

    如果在转储文件中的 GTID 集合之间存在或可能存在重叠的事务，则可以使用 Section 19.1.3.8, “操作 GTID 的存储函数示例”中描述的存储函数事先检查这一点，并计算所有 GTID 集合的并集。
