# 16.3 字典数据的事务性存储

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-dictionary-transactional-storage.html`](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-transactional-storage.html)

数据字典模式将字典数据存储在事务性（`InnoDB`）表中。数据字典表位于`mysql`数据库中，与非数据字典系统表一起。

数据字典表是在名为`mysql.ibd`的单个`InnoDB`表空间中创建的，该表空间位于 MySQL 数据目录中。`mysql.ibd`表空间文件必须位于 MySQL 数据目录中，其名称不能被修改或被其他表空间使用。

字典数据受到与保护存储在`InnoDB`表中的用户数据相同的提交、回滚和崩溃恢复功能的保护。
