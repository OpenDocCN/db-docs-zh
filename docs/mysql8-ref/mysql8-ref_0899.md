# 15.1.13 CREATE EVENT Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-event.html`](https://dev.mysql.com/doc/refman/8.0/en/create-event.html)

```sql
CREATE
    [DEFINER = *user*]
    EVENT
    [IF NOT EXISTS]
    *event_name*
    ON SCHEDULE *schedule*
    [ON COMPLETION [NOT] PRESERVE]
    [ENABLE | DISABLE | DISABLE ON SLAVE]
    [COMMENT '*string*']
    DO *event_body*;

*schedule*: {
    AT *timestamp* [+ INTERVAL *interval*] ...
  | EVERY *interval*
    [STARTS *timestamp* [+ INTERVAL *interval*] ...]
    [ENDS *timestamp* [+ INTERVAL *interval*] ...]
}

*interval*:
    *quantity* {YEAR | QUARTER | MONTH | DAY | HOUR | MINUTE |
              WEEK | SECOND | YEAR_MONTH | DAY_HOUR | DAY_MINUTE |
              DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}
```

此语句创建并安排了一个新事件。除非启用了事件调度程序，否则事件不会运行。有关检查事件调度程序状态并在必要时启用它的信息，请参阅 Section 27.4.2, “Event Scheduler Configuration”。

`CREATE EVENT`需要在要创建事件的模式中具有`EVENT`权限。如果存在`DEFINER`子句，则所需的权限取决于*`user`*值，如 Section 27.6, “Stored Object Access Control”中所讨论的那样。

`CREATE EVENT`语句的最低要求如下：

+   包含事件名称的关键字`CREATE EVENT`以及一个事件名称，该名称在数据库模式中唯一标识事件。

+   一个`ON SCHEDULE`子句，确定事件何时以及多久执行一次。

+   一个包含要由事件执行的 SQL 语句的`DO`子句。

这是一个最简单的`CREATE EVENT`语句的示例：

```sql
CREATE EVENT myevent
    ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
    DO
      UPDATE myschema.mytable SET mycol = mycol + 1;
```

上述语句创建了一个名为`myevent`的事件。此事件在创建后一小时执行一次，通过运行一个 SQL 语句来增加`myschema.mytable`表的`mycol`列的值。

*`event_name`*必须是一个有效的 MySQL 标识符，最大长度为 64 个字符。事件名称不区分大小写，因此在同一模式中不能有两个名为`myevent`和`MyEvent`的事件。一般来说，事件名称的规则与存储过程名称的规则相同。请参阅 Section 11.2, “Schema Object Names”。

事件与模式相关联。如果*`event_name`*的一部分未指定模式，则假定默认（当前）模式。要在特定模式中创建事件，请使用`*`schema_name`*.*`event_name`*`语法限定事件名称。

`DEFINER`子句指定在事件执行时检查访问权限时要使用的 MySQL 帐户。如果存在`DEFINER`子句，则*`user`*值应为 MySQL 帐户，指定为`'*`user_name`*'@'*`host_name`*'`，`CURRENT_USER`，或`CURRENT_USER()`。允许的*`user`*值取决于您拥有的权限，如 Section 27.6, “Stored Object Access Control”中所讨论的那样。还请参阅该部分以获取有关事件安全性的其他信息。

如果省略 `DEFINER` 子句，则默认的定义者是执行 `CREATE EVENT` 语句的用户。这与明确指定 `DEFINER = CURRENT_USER` 是相同的。

在事件体内，`CURRENT_USER` 函数返回用于在事件执行时检查权限的帐户，即 `DEFINER` 用户。有关事件内用户审计的信息，请参阅 第 8.2.23 节，“基于 SQL 的帐户活动审计”。

`IF NOT EXISTS` 对于 `CREATE EVENT` 与 `CREATE TABLE` 具有相同的含义：如果同一模式中已经存在名为 *`event_name`* 的事件，则不会执行任何操作，也不会产生错误。（但在这种情况下会生成警告。）

`ON SCHEDULE` 子句确定事件体 *`event_body`* 重复定义的事件何时、多久以及持续多久。此子句有两种形式之一：

+   `AT *`timestamp`*` 用于一次性事件。它指定事件仅在由 *`timestamp`* 给出的日期和时间执行一次，该时间戳必须包括日期和时间，或者必须是解析为日期时间值的表达式。你可以为此目的使用 `DATETIME` 或 `TIMESTAMP` 类型的值。如果日期已过去，将发出警告，如下所示：

    ```sql
    mysql> SELECT NOW();
    +---------------------+
    | NOW()               |
    +---------------------+
    | 2006-02-10 23:59:01 |
    +---------------------+
    1 row in set (0.04 sec)

    mysql> CREATE EVENT e_totals
     ->     ON SCHEDULE AT '2006-02-10 23:59:00'
     ->     DO INSERT INTO test.totals VALUES (NOW());
    Query OK, 0 rows affected, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Note
       Code: 1588
    Message: Event execution time is in the past and ON COMPLETION NOT
             PRESERVE is set. The event was dropped immediately after
             creation.
    ```

    `CREATE EVENT` 语句本身无效的情况下，无论出于何种原因，都会失败并显示错误。

    你可以使用 `CURRENT_TIMESTAMP` 来指定当前日期和时间。在这种情况下，事件在创建时立即执行。

    要创建一个在未来某个时间点发生的事件，相对于当前日期和时间，比如“从现在起三周”，你可以使用可选子句 `+ INTERVAL *`interval`*`。*`interval`* 部分由数量和时间单位组成，并遵循 时间间隔 中描述的语法规则，但在定义事件时不能使用涉及微秒的任何单位关键字。对于某些间隔类型，可以使用复杂的时间单位。例如，“两分钟十秒” 可以表示为 `+ INTERVAL '2:10' MINUTE_SECOND`。

    你也可以组合时间间隔。例如，`AT CURRENT_TIMESTAMP + INTERVAL 3 WEEK + INTERVAL 2 DAY` 相当于“从现在起三周零两天”。这样的子句中的每个部分都必须以 `+ INTERVAL` 开头。

+   要定期重复执行动作，使用 `EVERY` 子句。`EVERY` 关键字后面跟着一个如前面讨论的 `AT` 关键字描述的*`间隔`*。(`EVERY` 不与 `+ INTERVAL` 一起使用。) 例如，`EVERY 6 WEEK` 表示“每六周”。

    虽然 `+ INTERVAL` 子句在 `EVERY` 子句中不允许，但你可以使用与 `+ INTERVAL` 允许的相同复杂时间单位。

    `EVERY` 子句可以包含一个可选的 `STARTS` 子句。`STARTS` 后面跟着一个*`时间戳`*值，表示何时开始重复执行动作，并且还可以使用 `+ INTERVAL *`间隔`*` 来指定“从现在开始”的时间量。例如，`EVERY 3 MONTH STARTS CURRENT_TIMESTAMP + INTERVAL 1 WEEK` 表示“每三个月，从现在开始一周后”。同样，你可以表达“每两周，从现在开始六小时十五分钟后”为 `EVERY 2 WEEK STARTS CURRENT_TIMESTAMP + INTERVAL '6:15' HOUR_MINUTE`。不指定 `STARTS` 与使用 `STARTS CURRENT_TIMESTAMP` 是一样的——也就是说，事件指定的动作在事件创建时立即开始重复执行。

    `EVERY` 子句可以包含一个可选的 `ENDS` 子句。`ENDS` 关键字后面跟着一个*`时间戳`*值，告诉 MySQL 事件何时停止重复执行。你也可以在 `ENDS` 中使用 `+ INTERVAL *`间隔`*`；例如，`EVERY 12 HOUR STARTS CURRENT_TIMESTAMP + INTERVAL 30 MINUTE ENDS CURRENT_TIMESTAMP + INTERVAL 4 WEEK` 等同于“每十二小时，从现在开始三十分钟后，结束四周后”。不使用 `ENDS` 意味着事件会无限期执行。

    `ENDS` 支持与 `STARTS` 相同的复杂时间单位语法。

    在 `EVERY` 子句中可以使用 `STARTS`、`ENDS`、两者或者都不使用。

    如果重复事件在其调度间隔内没有终止，结果可能是同时执行多个事件实例。如果这是不希望的，你应该建立一个机制来防止同时执行实例。例如，你可以使用 `GET_LOCK()` 函数，或者行或表锁定。

`ON SCHEDULE` 子句可以使用涉及内置 MySQL 函数和用户变量的表达式来获取其中包含的任何*`时间戳`*或*`间隔`*值。在这样的表达式中，不能使用存储函数或可加载函数，也不能使用任何表引用；但是，你可以使用 `SELECT FROM DUAL`。这对于 `CREATE EVENT` 和 `ALTER EVENT` 语句都适用。在这种情况下，对存储函数、可加载函数和表的引用是明确不允许的，并会报错（参见 Bug #22830）。

`ON SCHEDULE`子句中的时间使用当前会话的`time_zone`值进行解释。这成为事件的时区；也就是说，用于事件调度并在事件执行时生效的时区。这些时间被转换为 UTC 并与事件时区一起存储在内部。这使得事件执行可以按照定义进行，而不受服务器时区或夏令时效果的任何后续更改的影响。有关事件时间表示的其他信息，请参见第 27.4.4 节，“事件元数据”。另请参见第 15.7.7.18 节，“SHOW EVENTS 语句”，以及第 28.3.14 节，“INFORMATION_SCHEMA EVENTS 表”。

通常，一旦事件过期，它将立即被删除。您可以通过指定`ON COMPLETION PRESERVE`来覆盖此行为。使用`ON COMPLETION NOT PRESERVE`仅仅是明确默认的非持久性行为。

您可以创建一个事件，但使用`DISABLE`关键字阻止其处于活动状态。或者，您可以使用`ENABLE`来明确默认状态，即活动状态。这在与`ALTER EVENT`（参见第 15.1.3 节，“ALTER EVENT 语句”）结合使用时最有用。

除了`ENABLE`或`DISABLE`之外，还可以出现第三个值；在副本上设置`DISABLE ON SLAVE`用于指示事件在副本上的状态，表示事件在复制源服务器上创建并复制到副本，但在副本上不执行。参见第 19.5.1.16 节，“调用功能的复制”。

使用`COMMENT`子句为事件提供评论。*`comment`*可以是最多 64 个字符的任何字符串，用于描述事件。评论文本作为字符串文字必须用引号括起来。

`DO`子句指定事件执行的操作，由 SQL 语句组成。几乎任何可以在存储过程中使用的有效 MySQL 语句也可以作为计划事件的操作语句。（参见第 27.8 节，“存储程序的限制”。）例如，以下事件`e_hourly`每小时一次从`site_activity`模式中的`sessions`表中删除所有行：

```sql
CREATE EVENT e_hourly
    ON SCHEDULE
      EVERY 1 HOUR
    COMMENT 'Clears out sessions table each hour.'
    DO
      DELETE FROM site_activity.sessions;
```

MySQL 在创建或更改事件时存储了`sql_mode`系统变量的设置，并始终以此设置执行事件，*无论事件开始执行时当前服务器 SQL 模式如何*。

包含在其`DO`子句中的`ALTER EVENT`语句的`CREATE EVENT`语句似乎成功；然而，当服务器尝试执行生成的计划事件时，执行将失败并出现错误。

注意

诸如`SELECT`或`SHOW`等仅返回结果集的语句在事件中使用时没有任何效果；这些语句的输出既不会发送到 MySQL 监视器，也不会存储在任何地方。但是，您可以使用诸如`SELECT ... INTO`和`INSERT INTO ... SELECT`等存储结果的语句。（请参见本节中后续示例中的一个实例。）

事件所属的模式是`DO`子句中表引用的默认模式。对其他模式中表的引用必须使用正确的模式名称进行限定。

与存储过程一样，您可以在`DO`子句中使用复合语句语法，通过使用`BEGIN`和`END`关键字，如下所示：

```sql
delimiter |

CREATE EVENT e_daily
    ON SCHEDULE
      EVERY 1 DAY
    COMMENT 'Saves total number of sessions then clears the table each day'
    DO
      BEGIN
        INSERT INTO site_activity.totals (time, total)
          SELECT CURRENT_TIMESTAMP, COUNT(*)
            FROM site_activity.sessions;
        DELETE FROM site_activity.sessions;
      END |

delimiter ;
```

本示例使用`delimiter`命令更改语句定界符。请参见第 27.1 节，“定义存储程序”。

在事件中可以使用更复杂的复合语句，例如存储过程中使用的语句。此示例使用了局部变量、错误处理程序和流程控制结构：

```sql
delimiter |

CREATE EVENT e
    ON SCHEDULE
      EVERY 5 SECOND
    DO
      BEGIN
        DECLARE v INTEGER;
        DECLARE CONTINUE HANDLER FOR SQLEXCEPTION BEGIN END;

        SET v = 0;

        WHILE v < 5 DO
          INSERT INTO t1 VALUES (0);
          UPDATE t2 SET s1 = s1 + 1;
          SET v = v + 1;
        END WHILE;
    END |

delimiter ;
```

无法直接传递参数到或从事件中；然而，在事件中调用带参数的存储过程是可能的：

```sql
CREATE EVENT e_call_myproc
    ON SCHEDULE
      AT CURRENT_TIMESTAMP + INTERVAL 1 DAY
    DO CALL myproc(5, 27);
```

如果事件的定义者具有足够设置全局系统变量的权限（参见第 7.1.9.1 节，“系统变量权限”），则事件可以读取和写入全局变量。由于授予此类权限可能导致滥用，因此在执行此操作时必须非常小心。

通常，存储过程中有效的任何语句都可以用于事件执行的操作语句。有关存储过程中允许的语句的更多信息，请参见第 27.2.1 节，“存储过程语法”。不可能将事件创建为存储过程的一部分，也不可能通过另一个事件创建事件。
