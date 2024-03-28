> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-databases.html`](https://dev.mysql.com/doc/refman/8.0/en/show-databases.html)

#### 15.7.7.14 显示数据库语句

```sql
SHOW {DATABASES | SCHEMAS}
    [LIKE '*pattern*' | WHERE *expr*]
```

`显示数据库` 列出了 MySQL 服务器主机上的数据库。 `显示模式` 是 `显示数据库` 的同义词。如果存在 `LIKE` 子句，则指示要匹配的数据库名称。可以使用 `WHERE` 子句选择使用更一般条件的行，如 第 28.8 节，“SHOW 语句的扩展” 中讨论的那样。

您只能看到您具有某种权限的数据库，除非具有全局 `显示数据库` 权限。您还可以使用 **mysqlshow** 命令获取此列表。

如果服务器是使用 `--skip-show-database` 选项启动的，则除非具有 `显示数据库` 权限，否则根本无法使用此语句。

MySQL 将数据库实现为数据目录中的目录，因此此语句仅列出该位置中的目录。但是，输出可能包括不对应实际数据库的目录名称。

数据库信息也可以从 `INFORMATION_SCHEMA` `SCHEMATA` 表中获取。请参阅 第 28.3.31 节，“INFORMATION_SCHEMA SCHEMATA 表”。

注意

因为任何静态全局权限都被视为所有数据库的权限，任何静态全局权限都使用户能够使用 `显示数据库` 或通过检查 `INFORMATION_SCHEMA` 的 `SCHEMATA` 表来查看所有数据库名称，除了通过部分撤销在数据库级别限制的数据库。
