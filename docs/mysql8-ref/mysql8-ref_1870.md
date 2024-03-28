# 27.2.3 存储过程元数据

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-routines-metadata.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-routines-metadata.html)

获取存储过程的元数据：

+   查询`INFORMATION_SCHEMA`数据库的`ROUTINES`表。参见 Section 28.3.30, “The INFORMATION_SCHEMA ROUTINES Table”。

+   使用`SHOW CREATE PROCEDURE`和`SHOW CREATE FUNCTION`语句查看过程定义。参见 Section 15.7.7.9, “SHOW CREATE PROCEDURE Statement”。

+   使用`SHOW PROCEDURE STATUS`和`SHOW FUNCTION STATUS`语句查看过程特征。参见 Section 15.7.7.28, “SHOW PROCEDURE STATUS Statement”。

+   使用`SHOW PROCEDURE CODE`和`SHOW FUNCTION CODE`语句查看过程的内部实现表示。参见 Section 15.7.7.27, “SHOW PROCEDURE CODE Statement”。
