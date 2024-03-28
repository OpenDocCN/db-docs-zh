> 原文：[`dev.mysql.com/doc/refman/8.0/en/delete-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/delete-optimization.html)

#### 10.2.5.3 优化 DELETE 语句

删除`MyISAM`表中单独行所需的时间与索引数量成正比。要更快地删除行，可以通过增加`key_buffer_size`系统变量的大小来增加键缓存的大小。参见 Section 7.1.1, “配置服务器”。

要从`MyISAM`表中删除所有行，`TRUNCATE TABLE *`tbl_name`*`比`DELETE FROM *`tbl_name`*`更快。截断操作不是事务安全的；在进行活动事务或活动表锁时尝试进行截断操作时会出现错误。参见 Section 15.1.37, “TRUNCATE TABLE 语句”。
