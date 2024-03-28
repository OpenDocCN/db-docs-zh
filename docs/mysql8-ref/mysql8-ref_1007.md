# 15.5.3 释放准备语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/deallocate-prepare.html`](https://dev.mysql.com/doc/refman/8.0/en/deallocate-prepare.html)

```sql
{DEALLOCATE | DROP} PREPARE *stmt_name*
```

要释放使用`PREPARE`生成的准备语句，使用一个引用准备语句名称的`DEALLOCATE PREPARE`语句。在释放准备语句后尝试执行它会导致错误。如果创建了太多准备语句，并且没有通过`DEALLOCATE PREPARE`语句或会话结束释放，您可能会遇到由`max_prepared_stmt_count`系统变量强制执行的上限。

有关示例，请参见第 15.5 节，“准备语句”。
