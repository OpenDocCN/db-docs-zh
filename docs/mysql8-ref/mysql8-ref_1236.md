# 17.11.5 使用 TRUNCATE TABLE 回收磁盘空间

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-truncate-table-reclaim-space.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-truncate-table-reclaim-space.html)

要在截断`InnoDB`表时回收操作系统磁盘空间，表必须存储在自己的.ibd 文件中。为了让表存储在自己的.ibd 文件中，在创建表时必须启用`innodb_file_per_table`。此外，被截断的表与其他表之间不能有外键约束，否则`TRUNCATE TABLE`操作会失败。然而，同一表中两列之间的外键约束是允许的。

当表被截断时，它会被删除并在一个新的`.ibd`文件中重新创建，释放的空间会返回给操作系统。这与截断存储在`InnoDB`系统表空间（当`innodb_file_per_table=OFF`时创建的表）和存储在共享通用表空间中的`InnoDB`表形成对比，在这种情况下，只有`InnoDB`在表被截断后才能使用释放的空间。

截断表并将磁盘空间返回给操作系统的能力也意味着物理备份可以更小。截断存储在系统表空间（当`innodb_file_per_table=OFF`时创建的表）或通用表空间中的表会在表空间中留下未使用的空间块。
