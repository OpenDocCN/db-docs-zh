> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-rule-based-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-rule-based-filtering.html)

#### 7.4.2.6 基于规则的错误日志过滤（log_filter_dragnet）

`log_filter_dragnet`日志过滤组件基于用户定义的规则进行日志过滤。

要启用`log_filter_dragnet`过滤器，首先加载过滤器组件，然后修改`log_error_services`的值。以下示例启用了与内置日志接收器结合使用的`log_filter_dragnet`：

```sql
INSTALL COMPONENT 'file://component_log_filter_dragnet';
SET GLOBAL log_error_services = 'log_filter_dragnet; log_sink_internal';
```

要设置`log_error_services`在服务器启动时生效，请使用第 7.4.2.1 节“错误日志配置”中的说明。这些说明也适用于其他错误日志系统变量。

启用`log_filter_dragnet`后，通过设置`dragnet.log_error_filter_rules`系统变量来定义其过滤规则。规则集由零个或多个规则组成，其中���个规则都是以句号（`.`）字符结尾的`IF`语句。如果变量值为空（零个规则），则不会进行过滤。

示例 1：此规则集丢弃信息事件，并对其他事件删除`source_line`字段：

```sql
SET GLOBAL dragnet.log_error_filter_rules =
  'IF prio>=INFORMATION THEN drop. IF EXISTS source_line THEN unset source_line.';
```

其效果类似于使用`log_error_verbosity=2`设置的`log_sink_internal`过滤器执行的过滤。

为了可读性，您可能会发现将规则列在单独的行上更可取。例如：

```sql
SET GLOBAL dragnet.log_error_filter_rules = '
  IF prio>=INFORMATION THEN drop.
  IF EXISTS source_line THEN unset source_line.
';
```

示例 2：此规则将信息事件限制为每 60 秒不超过一个：

```sql
SET GLOBAL dragnet.log_error_filter_rules =
  'IF prio>=INFORMATION THEN throttle 1/60.';
```

一旦您按照您的意愿设置了过滤配置，请考虑使用`SET PERSIST`而不是`SET GLOBAL`来分配`dragnet.log_error_filter_rules`，以使设置在服务器重新启动时持久化。或者，将设置添加到服务器选项文件中。

当使用`log_filter_dragnet`时，`log_error_suppression_list`会被忽略。

要停止使用过滤语言，首先从错误日志组件集中删除它。通常这意味着使用不同的过滤组件而不是没有过滤组件。例如：

```sql
SET GLOBAL log_error_services = 'log_filter_internal; log_sink_internal';
```

再次考虑使用`SET PERSIST`而不是`SET GLOBAL`来使设置在服务器重新启动时持久化。

然后卸载过滤器`log_filter_dragnet`组件：

```sql
UNINSTALL COMPONENT 'file://component_log_filter_dragnet';
```

以下部分更详细地描述了`log_filter_dragnet`操作的各个方面：

+   log_filter_dragnet 规则语言的语法

+   log_filter_dragnet 规则的操作

+   log_filter_dragnet 规则中的字段引用

##### log_filter_dragnet 规则语言的语法

以下语法定义了`log_filter_dragnet`过滤规则的语言。每个规则都是以句号（`.`）字符结尾的`IF`语句。该语言不区分大小写。

```sql
*rule*:
    IF *condition* THEN *action*
    [ELSEIF *condition* THEN *action*] ...
    [ELSE *action*]
    .

*condition*: {
    *field* *comparator* *value*
  | [NOT] EXISTS *field*
  | *condition* {AND | OR}  *condition*
}

*action*: {
    drop
  | throttle {*count* | *count* / *window_size*}
  | set *field* [:= | =] *value*
  | unset [*field*]
}

*field*: {
    *core_field*
  | *optional_field*
  | *user_defined_field*
}

*core_field*: {
    time
  | msg
  | prio
  | err_code
  | err_symbol
  | SQL_state
  | subsystem
}

*optional_field*: {
    OS_errno
  | OS_errmsg
  | label
  | user
  | host
  | thread
  | query_id
  | source_file
  | source_line
  | function
  | component
}

*user_defined_field*:
    *sequence of characters in [a-zA-Z0-9_] class*

*comparator*: {== | != | <> | >= | => | <= | =< | < | >}

*value*: {
    *string_literal*
  | *integer_literal*
  | *float_literal*
  | *error_symbol*
  | *priority*
}

*count*: *integer_literal*
*window_size*: *integer_literal*

*string_literal*:
    *sequence of characters quoted as '...' or "..."*

*integer_literal*:
    *sequence of characters in [0-9] class*

*float_literal*:
    *integer_literal*[.*integer_literal*]

*error_symbol*:
    *valid MySQL error symbol such as ER_ACCESS_DENIED_ERROR or ER_STARTUP*

*priority*: {
    ERROR
  | WARNING
  | INFORMATION
}
```

简单条件将字段与值进行比较或测试字段是否存在。要构建更复杂的条件，请使用`AND`和`OR`运算符。这两个运算符具有相同的优先级，并且从左到右进行评估。

要在字符串中转义字符，请在其前面加上反斜杠（`\`）。反斜杠用于包含反斜杠本身或字符串引号字符，对于其他字符是可选的。

为方便起见，`log_filter_dragnet`支持对某些字段进行比较的符号名称。为了可读性和可移植性，符号值比数值值更可取（适用的情况下）。

+   事件优先级值 1、2 和 3 可以指定为`ERROR`、`WARNING`和`INFORMATION`。优先级符号仅在与`prio`字段进行比较时才被识别。这些比较是等效的：

    ```sql
    IF prio == INFORMATION THEN ...
    IF prio == 3 THEN ...
    ```

+   错误代码可以以数字形式或相应的错误符号指定。例如，`ER_STARTUP`是错误`1408`的符号名称，因此这些比较是等效的：

    ```sql
    IF err_code == ER_STARTUP THEN ...
    IF err_code == 1408 THEN ...
    ```

    仅在与`err_code`字段和用户定义字段进行比较时才识别错误符号。

    要找到与给定错误代码号对应的错误符号，请使用以下方法之一：

    +   在服务器错误消息参考中查看服务器错误列表。

    +   使用**perror**命令。给定一个错误号参数，**perror**会显示有关错误的信息，包括其符号。

    假设一个带有错误号的规则集如下所示：

    ```sql
    IF err_code == 10927 OR err_code == 10914 THEN drop.
    IF err_code == 1131 THEN drop.
    ```

    使用**perror**，确定错误符号：

    ```sql
    $> perror 10927 10914 1131
    MySQL error code MY-010927 (ER_ACCESS_DENIED_FOR_USER_ACCOUNT_LOCKED):
    Access denied for user '%-.48s'@'%-.64s'. Account is locked.
    MySQL error code MY-010914 (ER_ABORTING_USER_CONNECTION):
    Aborted connection %u to db: '%-.192s' user: '%-.48s' host:
    '%-.64s' (%-.64s).
    MySQL error code MY-001131 (ER_PASSWORD_ANONYMOUS_USER):
    You are using MySQL as an anonymous user and anonymous users
    are not allowed to change passwords
    ```

    通过用数字替换错误符号，规则集变为：

    ```sql
    IF err_code == ER_ACCESS_DENIED_FOR_USER_ACCOUNT_LOCKED
      OR err_code == ER_ABORTING_USER_CONNECTION THEN drop.
    IF err_code == ER_PASSWORD_ANONYMOUS_USER THEN drop.
    ```

符号名称可以作为带引号的字符串指定，用于与字符串字段进行比较，但在这种情况下，名称是没有特殊含义的字符串，`log_filter_dragnet`不会将其解析为相应的数值。此外，拼写错误可能不会被检测到，而在尝试使用服务器不认识的未引用符号时，`SET`会立即出现错误。

##### `log_filter_dragnet` 规则的操作

`log_filter_dragnet` 在过滤规则中支持以下操作：

+   `drop`: 丢弃当前日志事件（不记录）。

+   `throttle`: 应用速率限制以减少匹配特定条件的事件的日志冗余。参数表示速率，形式为*`count`*或*`count`*/*`window_size`*。*`count`*值表示每个时间窗口允许记录的事件发生次数。*`window_size`*值是以秒为单位的时间窗口；如果省略，则默认窗口为 60 秒。这两个值必须是整数文字。

    此规则将插件关闭消息限制为每 60 秒 5 次：

    ```sql
    IF err_code == ER_PLUGIN_SHUTTING_DOWN_PLUGIN THEN throttle 5.
    ```

    此规则将错误和警告限制为每小时 1000 次，信息消息限制为每小时 100 次：

    ```sql
    IF prio <= INFORMATION THEN throttle 1000/3600 ELSE throttle 100/3600.
    ```

+   `set`: 为字段赋值（如果字段不存在，则创建）。在后续规则中，对字段名称的`EXISTS`测试为真，并且新值可以通过比较条件进行测试。

+   `unset`: 丢弃一个字段。在后续规则中，对字段名称的`EXISTS`测试为假，并且对字段与任何值的比较为假。

    在条件引用确切一个字段名称的特殊情况下，`unset`后面的字段名称是可选的，`unset`会丢弃命名字段。以下规则是等效的：

    ```sql
    IF myfield == 2 THEN unset myfield.
    IF myfield == 2 THEN unset.
    ```

##### `log_filter_dragnet` 规则中的字段引用

`log_filter_dragnet` 规则支持在错误事件中引用核心、可选和用户定义的字段。

+   核心字段引用

+   可选字段引用

+   用户定义字段引用

###### 核心字段引用

log_filter_dragnet 规则语言语法中列出了过滤规则识别的核心字段。有关这些字段的一般描述，请参见 7.4.2.3 节“错误事件字段”，假定您已经熟悉。以下备注仅提供与`log_filter_dragnet`规则中核心字段引用相关的特定信息。

+   `prio`

    事件优先级，用于指示错误、警告或注释/信息事件。在比较中，每个优先级可以指定为符号优先级名称或整数文字。优先级符号仅在与`prio`字段的比较中被识别。以下比较是等效的：

    ```sql
    IF prio == INFORMATION THEN ...
    IF prio == 3 THEN ...
    ```

    以下表格显示了允许的优先级级别。

    | 事件类型 | 优先级符号 | 数字优先级 |
    | --- | --- | --- |
    | 错误事件 | `ERROR` | 1 |
    | 警告事件 | `WARNING` | 2 |
    | 注意/信息事件 | `信息` | 3 |

    还有一个消息优先级为 `SYSTEM`，但系统消息无法被过滤，并且始终写入错误日志。

    优先级值遵循更高优先级具有较低值，反之亦然的原则。优先级值从最严重事件（错误）开始为 1，并随着优先级降低的事件而增加。例如，要丢弃优先级低于警告的事件，请测试高于 `WARNING` 的优先级值：

    ```sql
    IF prio > WARNING THEN drop.
    ```

    以下示例显示了实现与每个 `log_error_verbosity` 值类似效果的 `log_filter_dragnet` 规则：

    +   仅错误（`log_error_verbosity=1`）：

        ```sql
        IF prio > ERROR THEN drop.
        ```

    +   错误和警告（`log_error_verbosity=2`）：

        ```sql
        IF prio > WARNING THEN drop.
        ```

    +   错误、警告和注释（`log_error_verbosity=3`）：

        ```sql
        IF prio > INFORMATION THEN drop.
        ```

        实际上，这个规则可以被省略，因为没有比 `INFORMATION` 更大的 `prio` 值，因此实际上它什么也不丢弃。

+   `err_code`

    数字事件错误代码。在比较中，要测试的值可以指定为符号错误名称或整数文字。错误符号仅在与 `err_code` 字段和用户定义字段的比较中被识别。这些比较是等效的：

    ```sql
    IF err_code == ER_ACCESS_DENIED_ERROR THEN ...
    IF err_code == 1045 THEN ...
    ```

+   `err_symbol`

    事件错误符号，作为字符串（例如，`'ER_DUP_KEY'`）。 `err_symbol` 值更适用于识别日志输出中的特定行，而不适用于用于过滤规则比较，因为 `log_filter_dragnet` 不会将指定为字符串的比较值解析为等效的数值错误代码。（为了发生这种情况，必须使用未引用的符号指定错误。）

###### 可选字段引用

`log_filter_dragnet` 语法在 Grammar for log_filter_dragnet Rule Language 中命名了过滤规则识别的可选字段。有关这些字段的一般描述，请参见 Section 7.4.2.3, “Error Event Fields”，假定您已熟悉。以下备注仅提供与 `log_filter_dragnet` 规则中使用的可选字段引用相关的特定信息。

+   `标签`

    与 `prio` 值对应的标签，作为字符串。过滤规则可以更改支持自定义标签的日志接收器的标签。 `label` 值更适用于识别日志输出中的特定行，而不适用于用于过滤规则比较，因为 `log_filter_dragnet` 不会将指定为字符串的比较值解析为等效的数值优先级。

+   `source_file`

    事件发生的源文件，不包含任何前导路径。例如，要测试 `sql/gis/distance.cc` 文件，写比较如下：

    ```sql
    IF source_file == "distance.cc" THEN ...
    ```

###### 用户定义字段引用

在`log_filter_dragnet`过滤规则中，任何未被识别为核心或可选字段名称的字段名称都被视为用户定义字段。
