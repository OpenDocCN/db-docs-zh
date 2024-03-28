# 15.1.3 ALTER EVENT Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-event.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-event.html)

```sql
ALTER
    [DEFINER = *user*]
    EVENT *event_name*
    [ON SCHEDULE *schedule*]
    [ON COMPLETION [NOT] PRESERVE]
    [RENAME TO *new_event_name*]
    [ENABLE | DISABLE | DISABLE ON SLAVE]
    [COMMENT '*string*']
    [DO *event_body*]
```

`ALTER EVENT`语句更改现有事件的一个或多个特征，无需删除和重新创建。每个`DEFINER`、`ON SCHEDULE`、`ON COMPLETION`、`COMMENT`、`ENABLE` / `DISABLE`和`DO`子句的语法与与`CREATE EVENT`一起使用时完全相同。 （请参见第 15.1.13 节，“CREATE EVENT Statement”。）

任何用户都可以修改在其具有`EVENT`权限的数据库上定义的事件。当用户执行成功的`ALTER EVENT`语句时，该用户将成为受影响事件的定义者。

`ALTER EVENT`仅适用于现有事件：

```sql
mysql> ALTER EVENT no_such_event 
     >     ON SCHEDULE 
     >       EVERY '2:3' DAY_HOUR;
ERROR 1517 (HY000): Unknown event 'no_such_event'
```

在以下每个示例中，假设名为`myevent`的事件定义如下所示：

```sql
CREATE EVENT myevent
    ON SCHEDULE
      EVERY 6 HOUR
    COMMENT 'A sample comment.'
    DO
      UPDATE myschema.mytable SET mycol = mycol + 1;
```

以下语句将`myevent`的计划从立即开始的每六小时一次更改为从运行语句时开始的每十二小时一次，四小时后开始：

```sql
ALTER EVENT myevent
    ON SCHEDULE
      EVERY 12 HOUR
    STARTS CURRENT_TIMESTAMP + INTERVAL 4 HOUR;
```

可以在单个语句中更改事件的多个特征。此示例将`myevent`执行的 SQL 语句更改为删除`mytable`中的所有记录；还更改了事件的计划，使其在此`ALTER EVENT`语句运行后一天执行一次。

```sql
ALTER EVENT myevent
    ON SCHEDULE
      AT CURRENT_TIMESTAMP + INTERVAL 1 DAY
    DO
      TRUNCATE TABLE myschema.mytable;
```

仅为要更改的特征在`ALTER EVENT`语句中指定选项；省略的选项保留其现有值。这包括`CREATE EVENT`的任何默认值，如`ENABLE`。

要禁用`myevent`，请使用此`ALTER EVENT`语句：

```sql
ALTER EVENT myevent
    DISABLE;
```

`ON SCHEDULE`子句可以使用涉及内置 MySQL 函数和用户变量的表达式来获取其中包含的任何*`timestamp`*或*`interval`*值。您不能在这些表达式中使用存储过程或可加载函数，也不能使用任何表引用；但是，您可以使用`SELECT FROM DUAL`。这对`ALTER EVENT`和`CREATE EVENT`语句都适用。在这种情况下，对存储过程、可加载函数和表的引用是明确不允许的，并且会因错误而失败（请参见 Bug #22830）。

尽管包含另一个`ALTER EVENT`语句的`DO`子句的`ALTER EVENT`语句似乎成功了，但当服务器尝试执行生成的计划事件时，执行会因错误而失败。

要重命名事件，请使用`ALTER EVENT`语句的`RENAME TO`子句。此语句将事件`myevent`重命名为`yourevent`：

```sql
ALTER EVENT myevent
    RENAME TO yourevent;
```

您还可以使用`ALTER EVENT ... RENAME TO ...`和`*`db_name.event_name`*`表示法将事件移动到不同的数据库，如下所示：

```sql
ALTER EVENT olddb.myevent
    RENAME TO newdb.myevent;
```

要执行上述语句，执行它的用户必须在`olddb`和`newdb`数据库上都具有`EVENT`权限。

注意

没有`RENAME EVENT`语句。

值`DISABLE ON SLAVE`在副本上使用，而不是`ENABLE`或`DISABLE`，以指示在复制源服务器上创建并复制到副本的事件，在副本上不执行。通常，`DISABLE ON SLAVE`会根据需要自动设置；但是，在某些情况下，您可能希望或需要手动更改它。有关更多信息，请参见 Section 19.5.1.16, “Replication of Invoked Features”。
