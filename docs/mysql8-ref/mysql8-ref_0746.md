# 13.2.5 TIMESTAMP 和 DATETIME 的自动初始化和更新

> 原文：[`dev.mysql.com/doc/refman/8.0/en/timestamp-initialization.html`](https://dev.mysql.com/doc/refman/8.0/en/timestamp-initialization.html)

`TIMESTAMP` 和 `DATETIME` 列可以自动初始化并更新为当前日期和时间（即当前时间戳）。

对于表中的任何 `TIMESTAMP` 或 `DATETIME` 列，您可以将当前时间戳分配为默认值、自动更新值或两者：

+   自动初始化列对于未为该列指定值的插入行设置为当前时间戳。

+   自动更新列在行中的任何其他列的值从当前值更改时，将自动更新为当前时间戳。如果所有其他列都设置为它们的当前值，则自动更新列保持不变。要防止自动更新列在其他列更改时更新，请明确将其设置为当前值。即使其他列不更改，也要更新自动更新列，请明确将其设置为应具有的值（例如，将其设置为`CURRENT_TIMESTAMP`）。

此外，如果禁用了 `explicit_defaults_for_timestamp` 系统变量，则可以通过将其分配为 `NULL` 值来将任何 `TIMESTAMP`（但不是 `DATETIME`）列初始化或更新为当前日期和时间，除非已使用 `NULL` 属性定义了它以允许 `NULL` 值。

要指定自动属性，请在列定义中使用 `DEFAULT CURRENT_TIMESTAMP` 和 `ON UPDATE CURRENT_TIMESTAMP` 子句。子句的顺序无关紧要。如果列定义中同时存在两者，则任何一个都可以先出现。任何 `CURRENT_TIMESTAMP` 的同义词都与 `CURRENT_TIMESTAMP` 具有相同的含义。这些同义词包括 `CURRENT_TIMESTAMP()`、`NOW()`、`LOCALTIME`、`LOCALTIME()`、`LOCALTIMESTAMP` 和 `LOCALTIMESTAMP()`。

使用`DEFAULT CURRENT_TIMESTAMP`和`ON UPDATE CURRENT_TIMESTAMP`是特定于`TIMESTAMP` 和 `DATETIME` 的。`DEFAULT`子句也可以用于指定常量（非自动）默认值（例如，`DEFAULT 0` 或 `DEFAULT '2000-01-01 00:00:00'`）。

注意

以下示例使用`DEFAULT 0`，这是一个可能会产生警告或错误的默认值，具体取决于是否启用了严格的 SQL 模式或 `NO_ZERO_DATE` SQL 模式。请注意，`TRADITIONAL` SQL 模式包括严格模式和 `NO_ZERO_DATE`。请参阅 第 7.1.11 节，“服务器 SQL 模式”。

`TIMESTAMP` 或 `DATETIME` 列定义可以为默认值和自动更新值同时指定当前时间戳，其中一个指定当前时间戳，或者两者都不指定。不同列可以具有不同的自动属性组合。以下规则描述了可能性：

+   同时使用`DEFAULT CURRENT_TIMESTAMP`和`ON UPDATE CURRENT_TIMESTAMP`，列的默认值为当前时间戳，并且会自动更新为当前时间戳。

    ```sql
    CREATE TABLE t1 (
      ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      dt DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    );
    ```

+   使用`DEFAULT`子句但没有`ON UPDATE CURRENT_TIMESTAMP`子句，列具有给定的默认值，不会自动更新为当前时间戳。

    默认值取决于`DEFAULT`子句是否指定为`CURRENT_TIMESTAMP`或常量值。使用`CURRENT_TIMESTAMP`时，默认值为当前时间戳。

    ```sql
    CREATE TABLE t1 (
      ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
      dt DATETIME DEFAULT CURRENT_TIMESTAMP
    );
    ```

    使用常量时，默认值为给定值。在这种情况下，列根本没有自动属性。

    ```sql
    CREATE TABLE t1 (
      ts TIMESTAMP DEFAULT 0,
      dt DATETIME DEFAULT 0
    );
    ```

+   使用`ON UPDATE CURRENT_TIMESTAMP`子句和常量`DEFAULT`子句，列会自动更新为当前时间戳，并具有给定的常量默认值。

    ```sql
    CREATE TABLE t1 (
      ts TIMESTAMP DEFAULT 0 ON UPDATE CURRENT_TIMESTAMP,
      dt DATETIME DEFAULT 0 ON UPDATE CURRENT_TIMESTAMP
    );
    ```

+   使用`ON UPDATE CURRENT_TIMESTAMP`子句但没有`DEFAULT`子句，列会自动更新为当前时间戳，但不具有当前时间戳作为默认值。

    在这种情况下，默认值取决于类型。`TIMESTAMP` 的默认值为 0，除非使用`NULL`属性定义，此时默认值为`NULL`。

    ```sql
    CREATE TABLE t1 (
      ts1 TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,     -- default 0
      ts2 TIMESTAMP NULL ON UPDATE CURRENT_TIMESTAMP -- default NULL
    );
    ```

    `DATETIME` 的默认值为`NULL`，除非使用`NOT NULL`属性定义，此时默认值为 0。

    ```sql
    CREATE TABLE t1 (
      dt1 DATETIME ON UPDATE CURRENT_TIMESTAMP,         -- default NULL
      dt2 DATETIME NOT NULL ON UPDATE CURRENT_TIMESTAMP -- default 0
    );
    ```

`时间戳` 和 `日期时间` 列除非明确指定，否则没有自动属性，有一个例外：如果禁用了 `explicit_defaults_for_timestamp` 系统变量，则第一个 `时间戳` 列如果没有明确指定，则同时具有 `DEFAULT CURRENT_TIMESTAMP` 和 `ON UPDATE CURRENT_TIMESTAMP`。要抑制第一个 `时间戳` 列的自动属性，可以使用以下策略之一：

+   启用 `explicit_defaults_for_timestamp` 系统变量。在这种情况下，指定自动初始化和更新的 `DEFAULT CURRENT_TIMESTAMP` 和 `ON UPDATE CURRENT_TIMESTAMP` 子句是可用的，但除非明确包含在列定义中，否则不会分配给任何 `时间戳` 列。

+   或者，如果禁用了 `explicit_defaults_for_timestamp`，则执行以下操作之一：

    +   使用指定常量默认值的 `DEFAULT` 子句定义列。

    +   指定 `NULL` 属性。这也会导致列允许 `NULL` 值，这意味着您不能通过将列设置为 `NULL` 来分配当前时间戳。将 `NULL` 分配给列会将列设置为 `NULL`，而不是当前时间戳。要分配当前时间戳，请将列设置为 `CURRENT_TIMESTAMP` 或类似 `NOW()`。

考虑以下表定义：

```sql
CREATE TABLE t1 (
  ts1 TIMESTAMP DEFAULT 0,
  ts2 TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                ON UPDATE CURRENT_TIMESTAMP);
CREATE TABLE t2 (
  ts1 TIMESTAMP NULL,
  ts2 TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                ON UPDATE CURRENT_TIMESTAMP);
CREATE TABLE t3 (
  ts1 TIMESTAMP NULL DEFAULT 0,
  ts2 TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                ON UPDATE CURRENT_TIMESTAMP);
```

这些表具有以下属性：

+   在每个表定义中，第一个 `时间戳` 列没有自动初始化或更新。

+   这些表在处理 `NULL` 值的 `ts1` 列上有所不同。对于 `t1`，`ts1` 是 `NOT NULL`，将其赋值为 `NULL` 会将其设置为当前时间戳。对于 `t2` 和 `t3`，`ts1` 允许 `NULL`，将其赋值为 `NULL` 会将其设置为 `NULL`。

+   `t2` 和 `t3` 在 `ts1` 的默认值上有所不同。对于 `t2`，`ts1` 被定义为允许 `NULL`，因此在没有明确的 `DEFAULT` 子句的情况下，默认值也为 `NULL`。对于 `t3`，`ts1` 允许 `NULL` 但具有明确的默认值为 0。

如果 `时间戳` 或 `日期时间` 列定义在任何地方包含明确的小数秒精度值，则必须在整个列定义中使用相同的值。这是允许的：

```sql
CREATE TABLE t1 (
  ts TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6)
);
```

这是不允许的：

```sql
CREATE TABLE t1 (
  ts TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP(3)
);
```

#### 时间戳初始化和 NULL 属性

如果禁用了`explicit_defaults_for_timestamp`系统变量，则默认情况下`TIMESTAMP`列为`NOT NULL`，不能包含`NULL`值，并且将`NULL`分配给当前时间戳。要允许`TIMESTAMP`列包含`NULL`，请明确声明具有`NULL`属性。在这种情况下，默认值也变为`NULL`，除非使用指定不同默认值的`DEFAULT`子句覆盖。`DEFAULT NULL`可用于明确指定`NULL`作为默认值。（对于未声明具有`NULL`属性的`TIMESTAMP`列，`DEFAULT NULL`是无效的。）如果`TIMESTAMP`列允许`NULL`值，分配`NULL`会将其设置为`NULL`，而不是当前时间戳。

以下表包含几个允许`NULL`值的`TIMESTAMP`列：

```sql
CREATE TABLE t
(
  ts1 TIMESTAMP NULL DEFAULT NULL,
  ts2 TIMESTAMP NULL DEFAULT 0,
  ts3 TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP
);
```

允许`NULL`值的`TIMESTAMP`列在插入时*不会*自动获取当前时间戳，除非满足以下条件之一：

+   其默认值被定义为`CURRENT_TIMESTAMP`，并且未为列指定任何值。

+   `CURRENT_TIMESTAMP`或其任何同义词，如`NOW()`，被明确插入到列中。

换句话说，只有在允许`NULL`值的`TIMESTAMP`列的定义中包含`DEFAULT CURRENT_TIMESTAMP`时，才会自动初始化：

```sql
CREATE TABLE t (ts TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP);
```

如果`TIMESTAMP`列允许`NULL`值，但其定义不包括`DEFAULT CURRENT_TIMESTAMP`，则必须明确插入与当前日期和时间对应的值。假设表`t1`和`t2`具有以下定义：

```sql
CREATE TABLE t1 (ts TIMESTAMP NULL DEFAULT '0000-00-00 00:00:00');
CREATE TABLE t2 (ts TIMESTAMP NULL DEFAULT NULL);
```

要在任一表中的`TIMESTAMP`列设置为插入时的当前时间戳，请明确地分配该值。例如：

```sql
INSERT INTO t2 VALUES (CURRENT_TIMESTAMP);
INSERT INTO t1 VALUES (NOW());
```

如果启用了`explicit_defaults_for_timestamp`系统变量，则`TIMESTAMP`列只允许`NULL`值，如果声明了`NULL`属性。此外，`TIMESTAMP`列不允许将`NULL`分配给当前时间戳，无论是否声明了`NULL`或`NOT NULL`属性。要分配当前时间戳，请将列设置为`CURRENT_TIMESTAMP`或类似的同义词，如`NOW()`。
