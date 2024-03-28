# A.2 MySQL 8.0 FAQ：存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-storage-engines.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-storage-engines.html)

A.2.1\. 我在哪里可以获取 MySQL 存储引擎的完整文档？

A.2.2\. MySQL 8.0 中是否有任何新的存储引擎？

A.2.3\. MySQL 8.0 中是否移除了任何存储引擎？

A.2.4\. 我可以阻止使用特定的存储引擎吗？

A.2.5\. 仅使用 InnoDB 存储引擎是否比使用 InnoDB 和非 InnoDB 存储引擎的组合有优势？

A.2.6\. ARCHIVE 存储引擎有哪些独特的优势？

| **A.2.1.** | 我在哪里可以获取 MySQL 存储引擎的完整文档？ |
| --- | --- |
|  | 请参阅 第十八章，“替代存储引擎”。该章节包含了除 `InnoDB` 存储引擎和 `NDB` 存储引擎（用于 MySQL Cluster）之外的所有 MySQL 存储引擎的信息。`InnoDB` 在 第十七章，“InnoDB 存储引擎” 中有介绍。`NDB` 在 第二十五章，“MySQL NDB Cluster 8.0” 中有介绍。 |
| **A.2.2.** | MySQL 8.0 中是否有任何新的存储引擎？ |
|  | 没有。`InnoDB` 是新表的默认存储引擎。有关详细信息，请参阅 第 17.1 节，“InnoDB 简介”。 |
| **A.2.3.** | MySQL 8.0 中是否移除了任何存储引擎？ |
|  | 提供分区支持的 `PARTITION` 存储引擎插件被本机分区处理程序取代。作为这一变化的一部分，服务器不能再使用 `-DWITH_PARTITION_STORAGE_ENGINE` 构建。`partition` 也不再显示在 `SHOW PLUGINS` 的输出中，或显示在 `INFORMATION_SCHEMA.PLUGINS` 表中。为了支持给定表的分区，现在用于表的存储引擎必须提供自己的（“本机”）分区处理程序。`InnoDB` 是 MySQL 8.0 中唯一支持本机分区处理程序的存储引擎。在 MySQL 8.0 中使用任何其他存储引擎创建分区表的尝试将失败。（MySQL Cluster 使用的 `NDB` 存储引擎也提供自己的分区处理程序，但目前不受 MySQL 8.0 支持。） |
| **A.2.4.** | 我可以阻止使用特定存储引擎吗？ |
|  | 是的。`disabled_storage_engines` 配置选项定义了哪些存储引擎不能用于创建表或表空间。默认情况下，`disabled_storage_engines` 为空（没有禁用的引擎），但可以设置为一个或多个引擎的逗号分隔列表。 |
| **A.2.5.** | 仅使用 `InnoDB` 存储引擎有什么优势，与使用 `InnoDB` 和非 `InnoDB` 存储引擎的组合相比？ |
|  | 是的。仅使用 `InnoDB` 表可以简化备份和恢复操作。MySQL 企业版备份对所有使用 `InnoDB` 存储引擎的表进行热备份。对于使用 `MyISAM` 或其他非 `InnoDB` 存储引擎的表，它进行“温暖”备份，数据库继续运行，但这些表在备份时无法修改。参见第 32.1 节，“MySQL 企业版备份概述”。 |
| **A.2.6.** | `ARCHIVE` 存储引擎的独特优势是什么？ |
|  | `ARCHIVE` 存储引擎存储大量数据而不使用索引；它占用空间小，并使用表扫描执行选择操作。详细信息请参见第 18.5 节，“ARCHIVE 存储引擎”。 |
