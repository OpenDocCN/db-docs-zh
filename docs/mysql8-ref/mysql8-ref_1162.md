> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-migration.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-migration.html)

#### 17.6.1.4 移动或复制 InnoDB 表

本节描述了将一些或所有 `InnoDB` 表移动或复制到不同服务器或实例的技术。例如，您可能会将整个 MySQL 实例移动到更大、更快的服务器；您可能会克隆整个 MySQL 实例到新的复制服务器；您可能会将单个表复制到另一个实例以开发和测试应用程序，或者将其复制到数据仓库服务器以生成报告。

在 Windows 上，`InnoDB` 总是在内部以小写存储数据库和表名。要将数据库以二进制格式从 Unix 移动到 Windows 或从 Windows 移动到 Unix，请使用小写名称创建所有数据库和表。实现这一目标的一种便捷方法是在创建任何数据库或表之前将以下行添加到您的 `my.cnf` 或 `my.ini` 文件的 `[mysqld]` 部分：

```sql
[mysqld]
lower_case_table_names=1
```

注意

禁止使用与服务器初始化时使用的设置不同的 `lower_case_table_names` 设置启动服务器。

移动或复制 `InnoDB` 表的技术包括：

+   导入表

+   MySQL Enterprise Backup

+   复制数据文件（冷备份方法）

+   从逻辑备份恢复

##### 导入表

位于文件表空间中的表可以使用 *可传输表空间* 功能从另一个 MySQL 服务器实例或备份中导入。请参阅 第 17.6.1.3 节，“导入 InnoDB 表”。

##### MySQL Enterprise Backup

MySQL Enterprise Backup 产品可让您在最小干扰运营的情况下备份正在运行的 MySQL 数据库，并生成数据库的一致快照。当 MySQL Enterprise Backup 复制表时，读写可以继续进行。此外，MySQL Enterprise Backup 可以创建压缩备份文件，并备份表的子集。结合 MySQL 二进制日志，您可以执行按时间点恢复。MySQL Enterprise Backup 包含在 MySQL Enterprise 订阅的一部分中。

有关 MySQL Enterprise Backup 的更多详细信息，请参阅 第 32.1 节，“MySQL Enterprise Backup 概述”。

##### 复制数据文件（冷备份方法）

您可以通过复制 第 17.18.1 节，“InnoDB 备份” 中列出的所有相关文件来移动 `InnoDB` 数据库。

在所有具有相同浮点数格式的平台上，`InnoDB`数据和日志文件是二进制兼容的。如果浮点格式不同，但你的表中没有使用`FLOAT` - FLOAT, DOUBLE")或`DOUBLE` - FLOAT, DOUBLE")数据类型，则程序是相同的：只需复制相关文件。

当移动或复制基于文件的表格`.ibd`文件时，源和目标系统的数据库目录名称必须相同。存储在`InnoDB`共享表空间中的表定义包括数据库名称。表空间文件中存储的事务 ID 和日志序列号也在不同的数据库之间有所不同。

要将一个`.ibd`文件和相关表从一个数据库移动到另一个数据库，使用`RENAME TABLE`语句：

```sql
RENAME TABLE *db1.tbl_name* TO *db2.tbl_name*;
```

如果你有一个“干净”的`.ibd`文件备份，你可以按照以下步骤将其恢复到它原来的 MySQL 安装中：

1.  自从你复制`.ibd`文件以来，表不能被删除或截断，因为这样会改变存储在表空间内部的表 ID。

1.  发出这个`ALTER TABLE`语句来删除当前的`.ibd`文件：

    ```sql
    ALTER TABLE *tbl_name* DISCARD TABLESPACE;
    ```

1.  将备份的`.ibd`文件复制到正确的数据库目录中。

1.  发出这个`ALTER TABLE`语句，告诉`InnoDB`使用新的`.ibd`文件来替换表：

    ```sql
    ALTER TABLE *tbl_name* IMPORT TABLESPACE;
    ```

    注意

    `ALTER TABLE ... IMPORT TABLESPACE`功能不会对导入的数据强制执行外键约束。

在这种情况下，“干净”的`.ibd`文件备份是指满足以下要求的备份：

+   `.ibd`文件中没有事务中未提交的修改。

+   `.ibd`文件中没有未合并的插入缓冲区条目。

+   清除已从`.ibd`文件中删除标记的索引记录。

+   **mysqld**已经将`.ibd`文件的所有修改页面从缓冲池刷新到文件中。

你可以使用以下方法制作一个干净的备份`.ibd`文件：

1.  停止所有来自**mysqld**服务器��活动，并提交所有事务。

1.  等到`SHOW ENGINE INNODB STATUS`显示数据库中没有活动事务，并且`InnoDB`的主线程状态为`等待服务器活动`。然后你可以复制`.ibd`文件。

制作一个`.ibd`文件的干净副本的另一种方法是使用 MySQL 企业备份产品：

1.  使用 MySQL 企业备份来备份`InnoDB`安装。

1.  在备份上启动第二个**mysqld**服务器，并让它清理备份中的`.ibd`文件。

##### 从逻辑备份中恢复

你可以使用类似 **mysqldump** 这样的实用程序执行逻辑备份，它会生成一组可以执行的 SQL 语句，以便在转移到另一个 SQL 服务器时重新生成原始数据库对象定义和表数据。使用这种方法，无论格式是否不同或者你的表是否包含浮点数据都无关紧要。

为了提高此方法的性能，在导入数据时禁用 `autocommit`。只有在导入整个表或表的一部分后才执行提交。
