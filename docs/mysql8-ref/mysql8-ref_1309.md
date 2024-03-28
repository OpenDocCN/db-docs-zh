> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-table-close.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-table-close.html)

#### 18.2.4.2 未正确关闭表导致的问题

每个`MyISAM`索引文件（`.MYI`文件）的头部都有一个计数器，可用于检查表是否已正确关闭。如果您从`CHECK TABLE`或**myisamchk**收到以下警告，则表示该计数器已经不同步：

```sql
clients are using or haven't closed the table properly
```

此警告并不一定意味着表已损坏，但您至少应该检查表。

该计数器的工作方式如下：

+   在 MySQL 中第一次更新表时，索引文件头部的计数器会递增。

+   在进一步更新期间，计数器不会更改。

+   当表的最后一个实例关闭时（因为执行了`FLUSH TABLES`操作或因为表缓存中没有空间），如果表在任何时候已更新，则计数器会递减。

+   当修复表或检查表并发现表正常时，计数器将被重置为零。

+   为避免与可能检查表的其他进程交互的问题，如果计数器为零，则在关闭时不会减少。

换句话说，只有在以下情况下计数器才会变得不正确：

+   一个`MyISAM`表在未首先执行`LOCK TABLES`和`FLUSH TABLES`的情况下被复制。

+   MySQL 在更新和最终关闭之间崩溃。（表可能仍然正常，因为 MySQL 总是在每个语句之间为所有内容发出写入。）

+   一个表在被**myisamchk --recover**或**myisamchk --update-state**修改时，同时被**mysqld**使用。

+   多个**mysqld**服务器正在使用该表，其中一个服务器在另一个服务器使用该表时执行了`REPAIR TABLE`或`CHECK TABLE`。在这种设置中，使用`CHECK TABLE`是安全的，尽管您可能会收到其他服务器的警告。但是，应避免使用`REPAIR TABLE`，因为当一个服务器用新文件替换数据文件时，其他服务器不知道这一点。

    一般来说，将数据目录共享给多台服务器是一个不好的主意。参见第 7.8 节，“在一台机器上运行多个 MySQL 实例”，进行进一步讨论。
