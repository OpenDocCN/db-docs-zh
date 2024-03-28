# 6.8.2 perror — 显示 MySQL 错误消息信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/perror.html`](https://dev.mysql.com/doc/refman/8.0/en/perror.html)

**perror** 显示 MySQL 或操作系统错误代码的错误消息。像这样调用 **perror**：

```sql
perror [*options*] *errorcode* ...
```

**perror** 尝试灵活地理解其参数。例如，对于 `ER_WRONG_VALUE_FOR_VAR` 错误，**perror** 理解任何这些参数：`1231`，`001231`，`MY-1231`，或 `MY-001231`，或 `ER_WRONG_VALUE_FOR_VAR`。

```sql
$> perror 1231
MySQL error code MY-001231 (ER_WRONG_VALUE_FOR_VAR): Variable '%-.64s'
can't be set to the value of '%-.200s'
```

如果错误编号在 MySQL 和操作系统错误重叠的范围内，**perror** 将显示两个错误消息：

```sql
$> perror 1 13
OS error code   1:  Operation not permitted
MySQL error code MY-000001: Can't create/write to file '%s' (OS errno %d - %s)
OS error code  13:  Permission denied
MySQL error code MY-000013: Can't get stat of '%s' (OS errno %d - %s)
```

要获取 MySQL Cluster 错误代码的错误消息，请使用 **ndb_perror** 实用程序。

系统错误消息的含义可能取决于您的操作系统。在不同操作系统上，给定的错误代码可能有不同的含义。

**perror** 支持以下选项。

+   `--help`, `--info`, `-I`, `-?`

    显示帮助消息并退出。

+   `--ndb`

    打印 MySQL Cluster 错误代码的错误消息。

    此选项已在 MySQL 8.0.13 中移除。请改用 **ndb_perror** 实用程序。

+   `--silent`, `-s`

    静默模式。仅打印错误消息。

+   `--verbose`, `-v`

    详细模式。打印错误代码和消息。这是默认行为。

+   `--version`, `-V`

    显示版本信息并退出。
