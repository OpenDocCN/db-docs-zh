# 15.3.4 保存点、回滚到保存点和释放保存点语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/savepoint.html`](https://dev.mysql.com/doc/refman/8.0/en/savepoint.html)

```sql
SAVEPOINT *identifier*
ROLLBACK [WORK] TO [SAVEPOINT] *identifier*
RELEASE SAVEPOINT *identifier*
```

`InnoDB`支持 SQL 语句`SAVEPOINT`、`ROLLBACK TO SAVEPOINT`、`RELEASE SAVEPOINT`以及用于`ROLLBACK`的可选`WORK`关键字。

`SAVEPOINT`语句使用*`identifier`*设置具有名称的事务保存点。如果当前事务具有相同名称的保存点，则旧保存点将被删除，并设置一个新的保存点。

`ROLLBACK TO SAVEPOINT`语句将事务回滚到指定的保存点，而不终止事务。在回滚中，当前事务在设置保存点后对行所做的修改将被撤消，但`InnoDB`不会释放保存点后存储在内存中的行锁。（对于新插入的行，锁信息由存储在行中的事务 ID 携带；锁不会单独存储在内存中。在这种情况下，行锁在撤消时被释放。）比指定保存点设置的保存点将被删除。

如果`ROLLBACK TO SAVEPOINT`语句返回以下错误，则表示不存在具有指定名称的保存点：

```sql
ERROR 1305 (42000): SAVEPOINT *identifier* does not exist
```

`RELEASE SAVEPOINT`语句从当前事务的保存点集中移除指定的保存点。不会发生提交或回滚。如果保存点不存在，则会报错。

如果执行`COMMIT`或未命名保存点的`ROLLBACK`，则会删除当前事务的所有保存点。

当调用存储函数或触发器被激活时，将创建一个新的保存点级别。之前级别上的保存点变得不可用，因此不会与新级别上的保存点发生冲突。当函数或触发器终止时，它创建的任何保存点都会被释放，并且之前的保存点级别会被恢复。
