# 15.6.5 流程控制语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/flow-control-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/flow-control-statements.html)

15.6.5.1 CASE 语句

15.6.5.2 IF 语句

15.6.5.3 ITERATE 语句

15.6.5.4 LEAVE 语句

15.6.5.5 LOOP 语句

15.6.5.6 REPEAT 语句

15.6.5.7 RETURN 语句

15.6.5.8 WHILE 语句

MySQL 支持 `IF`、`CASE`、`ITERATE`、`LEAVE`、`LOOP`、`WHILE` 和 `REPEAT` 结构用于存储过程中的流程控制。它还支持存储函数中的 `RETURN`。

许多这些结构包含其他语句，如下一节中的语法规范所示。这些结构可以嵌套。例如，一个 `IF` 语句可能包含一个 `WHILE` 循环，而该循环本身包含一个 `CASE` 语句。

MySQL 不支持 `FOR` 循环。
