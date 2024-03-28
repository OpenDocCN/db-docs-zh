# A.8 MySQL 8.0 FAQ：迁移

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-migration.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-migration.html)

A.8.1\. 我在哪里可以找到有关如何从 MySQL 5.7 迁移到 MySQL 8.0 的信息？

A.8.2\. MySQL 8.0 中存储引擎（表类型）支持与以前版本有何变化？

| **A.8.1.** | 我在哪里可以找到有关如何从 MySQL 5.7 迁移到 MySQL 8.0 的信息？ |
| --- | --- |
|  | 有关详细的升级信息，请参阅 第三章，*升级 MySQL*。在升级时不要跳过一个主要版本，而是逐步完成过程，每次从一个主要版本升级到下一个主要版本。这可能看起来更复杂，但最终可以节省时间和麻烦。如果在升级过程中遇到问题，其来源更容易识别，无论是您自己还是如果您有 MySQL Enterprise 订阅，则由 MySQL 支持。 |
| **A.8.2.** | MySQL 8.0 中存储引擎（表类型）支持与以前版本有何变化？ |

|  | 存储引擎支持已更改如下：

+   MySQL 5.0 中移除了对 `ISAM` 表的支持，现在应该使用 `MyISAM` 存储引擎代替 `ISAM`。要将表 *`tblname`* 从 `ISAM` 转换为 `MyISAM`，只需发出类似于以下语句：

    ```sql
    ALTER TABLE *tblname* ENGINE=MYISAM;
    ```

+   MySQL 5.0 中也移除了 `MyISAM` 表的内部 `RAID`。以前用于允许在不支持大于 2GB 文件大小的文件系统中的大表。所有现代文件系统都允许更大的表；此外，现在还有其他解决方案，如 `MERGE` 表和视图。

+   现在，`VARCHAR` 列类型在所有存储引擎中保留尾随空格。

+   `MEMORY` 表（以前称为 `HEAP` 表）也可以包含 `VARCHAR` 列。

|
