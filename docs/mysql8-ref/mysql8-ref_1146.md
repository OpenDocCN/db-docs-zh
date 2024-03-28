# 17.1.2 InnoDB 表的最佳实践

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-best-practices.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-best-practices.html)

本节描述了使用`InnoDB`表时的最佳实践。

+   为每个表指定一个主键，使用最常查询的列或列，或者如果没有明显的主键，则使用自增值。

+   使用连接（joins）在从多个表中提取数据时，基于这些表中相同 ID 值进行连接。为了获得快速的连接性能，在连接列上定义外键，并在每个表中声明具有相同数据类型的列。添加外键可以确保引用列被索引，从而提高性能。外键还会将删除和更新传播到所有受影响的表，并防止在子表中插入数据，如果相应的 ID 在父表中不存在。

+   关闭自动提交。每秒提交数百次会限制性能（受存储设备的写入速度限制）。

+   将相关的 DML 操作组合成事务，通过使用`START TRANSACTION`和`COMMIT`语句将它们括起来。虽然您不希望经常提交，但也不希望发出运行数小时而不提交的大批量`INSERT`、`UPDATE`或`DELETE`语句。

+   不要使用`LOCK TABLES`语句。`InnoDB`可以处理多个会话同时读写同一张表，而不会牺牲可靠性或高性能。要获得对一组行的独占写访问权限，请使用`SELECT ... FOR UPDATE`语法，仅锁定您打算更新的行。

+   启用`innodb_file_per_table`变量或使用通用表空间将表的数据和索引放入单独的文件中，而不是系统表空间。`innodb_file_per_table`变量默认启用。

+   评估您的数据和访问模式是否受益于`InnoDB`表或页面压缩功能。您可以压缩`InnoDB`表而不会牺牲读/写能力。

+   使用`--sql_mode=NO_ENGINE_SUBSTITUTION`选项运行服务器，以防止使用您不想使用的存储引擎创建表。
