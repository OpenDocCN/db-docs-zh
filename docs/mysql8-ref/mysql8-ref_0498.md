> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-stored-programs.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-stored-programs.html)

#### 9.4.5.3 存储程序的导出

控制**mysqldump**如何处理存储程序（存储过程和函数，触发器和事件）的几个选项：

+   `--events`: 导出事件调度器事件

+   `--routines`: 导出存储过程和函数

+   `--triggers`: 为表导出触发器

`--triggers`选项默认启用，因此在导出表时，它们将附带任何触发器。其他选项默认禁用，必须明确指定以导出相应的对象。要明确禁用这些选项中的任何一个，请使用其跳过形式：`--skip-events`，`--skip-routines`，或`--skip-triggers`。
