# 32.1 MySQL 企业备份概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-backup.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-backup.html)

MySQL 企业备份执行 MySQL 数据库的热备份操作。该产品的架构旨在对由 InnoDB 存储引擎创建的表进行高效可靠的备份。为了完整性，它还可以备份来自 MyISAM 和其他存储引擎的表。

以下讨论简要总结了 MySQL 企业备份。有关更多信息，请参阅 MySQL 企业备份手册，网址为`dev.mysql.com/doc/mysql-enterprise-backup/en/`。

热备份是在数据库运行并且应用程序正在读写时执行的。这种类型的备份不会阻塞正常的数据库操作，并且即使在备份过程中发生更改，也会捕获这些更改。出于这些原因，当您的数据库“成熟”时，即数据足够大以至于备份需要显著时间，并且您的数据对您的业务非常重要以至于您必须捕获每一个变化而不使您的应用程序、网站或网络服务下线时，热备份是可取的。

MySQL 企业备份对使用 InnoDB 存储引擎的所有表进行热备份。对于使用 MyISAM 或其他非 InnoDB 存储引擎的表，它进行“温暖”备份，数据库继续运行，但在备份过程中这些表无法修改。为了进行高效的备份操作，您可以将 InnoDB 指定为新表的默认存储引擎，或将现有表转换为使用 InnoDB 存储引擎。
