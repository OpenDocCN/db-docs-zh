# 30.1 使用 sys 模式的先决条件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-schema-prerequisites.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-prerequisites.html)

在使用`sys`模式之前，必须满足本节中描述的先决条件。

因为`sys`模式提供了访问性能模式的另一种方式，所以性能模式必须启用才能使`sys`模式正常工作。请参阅第 29.3 节“性能模式启动配置”。

要完全访问`sys`模式，用户必须具有以下权限：

+   对所有`sys`表和视图进行`SELECT`操作

+   对所有`sys`存储过程和函数进行`EXECUTE`操作

+   如果要对其进行更改，则对`sys_config`表进行`INSERT`和`UPDATE`操作

+   对于某些`sys`模式存储过程和函数，还需要额外的权限，如其描述中所述（例如，`ps_setup_save()`过程")过程）

还必须具有底层对象的权限，这些对象是`sys`模式对象的基础：

+   对由`sys`模式对象访问的任何性能模式表进行`SELECT`操作，并且对使用`sys`模式对象更新的任何表进行`UPDATE`操作

+   对`INFORMATION_SCHEMA`中的`INNODB_BUFFER_PAGE`表进行`PROCESS`操作

必须启用某些性能模式工具和消费者，并且（对于工具）必须定时以充分利用`sys`模式的功能：

+   所有`wait`工具

+   所有`stage`工具

+   所有`statement`工具

+   `*`xxx`*_current`和`*`xxx`*_history_long`消费者适用于所有事件

你可以使用`sys`模式本身来启用所有额外的工具和消费者：

```sql
CALL sys.ps_setup_enable_instrument('wait');
CALL sys.ps_setup_enable_instrument('stage');
CALL sys.ps_setup_enable_instrument('statement');
CALL sys.ps_setup_enable_consumer('current');
CALL sys.ps_setup_enable_consumer('history_long');
```

注意

对于`sys`模式的许多用途，默认的性能模式已足以进行数据收集。启用所有上述提到的仪器和消费者会对性能产生影响，因此最好只启用您需要的额外配置。另外，请记住，如果您启用了额外的配置，可以轻松地像这样恢复默认配置：

```sql
CALL sys.ps_setup_reset_to_default(TRUE);
```
