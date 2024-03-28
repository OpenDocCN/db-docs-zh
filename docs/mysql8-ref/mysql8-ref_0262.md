> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-configuration.html)

#### 7.4.2.1 错误日志配置

在 MySQL 8.0 中，错误日志记录使用了在第 7.5 节，“MySQL 组件”中描述的 MySQL 组件架构。错误日志子系统由执行日志事件过滤和写入的组件以及配置哪些组件加载和启用以实现所需日志记录结果的系统变量组成。

本节讨论了如何加载和启用错误日志记录的组件。有关特定于日志过滤器的说明，请参阅第 7.4.2.4 节，“错误日志过滤类型”。有关特定于 JSON 和系统日志接收器的说明，请参阅第 7.4.2.7 节，“以 JSON 格式记录错误日志”和第 7.4.2.8 节，“将错误日志记录到系统日志”。有关所有可用日志组件的更多详细信息，请参阅第 7.5.3 节，“错误日志组件”。

基于组件的错误日志记录提供了以下功能：

+   可由过滤组件过滤的日志事件，以影响可用于写入的信息。

+   由接收器（写入器）组件输出的日志事件。可以启用多个接收器组件，将错误日志输出写入多个目的地。

+   实现默认错误日志格式的内置过滤器和接收器组件。

+   一个可加载的接收器，可启用以 JSON 格式记录日志。

+   一个可加载的接收器，可启用将日志记录到系统日志中。

+   控制加载和启用哪些日志组件以及每个组件如何运行的系统变量。

错误日志配置在本节中描述如下主题：

+   默认错误日志配置

+   错误日志配置方法

+   隐式错误日志配置

+   显式错误日志配置

+   更改错误日志配置方法

+   故障排除配置问题

+   配置多个日志接收器

+   日志汇流性能模式支持

##### 默认错误日志配置

`log_error_services`系统变量控制要加载的可加载日志组件（从 MySQL 8.0.30 开始）以及要为错误日志记录启用的日志组件。默认情况下，`log_error_services`具有以下值：

```sql
mysql> SELECT @@GLOBAL.log_error_services;
+----------------------------------------+
| @@GLOBAL.log_error_services            |
+----------------------------------------+
| log_filter_internal; log_sink_internal |
+----------------------------------------+
```

该值表示日志事件首先通过`log_filter_internal`过滤组件，然后通过`log_sink_internal`汇流组件，这两个都是内置组件。过滤器修改后续命名的组件看到的日志事件。汇流是日志事件的目的地。通常，汇流将日志事件处理为具有特定格式的日志消息，并将这些消息写入其关联的输出，例如文件或系统日志。

`log_filter_internal`和`log_sink_internal`的组合实现了默认的错误日志过滤和输出行为。这些组件的操作受其他服务器选项和系统变量的影响：

+   输出目的地由`--log-error`选项确定（在 Windows 上，还有`--pid-file`和`--console`）。这些选项确定是否将错误消息写入控制台或文件，如果写入文件，则确定错误日志文件名。参见 Section 7.4.2.2, “Default Error Log Destination Configuration”。

+   `log_error_verbosity`和`log_error_suppression_list`系统变量影响`log_filter_internal`允许或抑制哪些类型的日志事件。参见 Section 7.4.2.5, “Priority-Based Error Log Filtering (log_filter_internal)”")。

在配置`log_error_services`时，请注意以下特性：

+   日志组件列表可以用分号或（从 MySQL 8.0.12 开始）逗号分隔，可选地跟随空格。给定设置不能同时使用分号和逗号分隔符。组件顺序很重要，因为服务器按照列出的顺序执行组件。

+   `log_error_services`值中的最终组件不能是过滤器。这是一个错误，因为它对事件的任何更改都不会对输出产生影响：

    ```sql
    mysql> SET GLOBAL log_error_services = 'log_filter_internal';
    ERROR 1231 (42000): Variable 'log_error_services' can't be set to the value
    of 'log_filter_internal'
    ```

    要纠正问题，请在值的末尾包含一个 sink：

    ```sql
    mysql> SET GLOBAL log_error_services = 'log_filter_internal; log_sink_internal';
    ```

+   在 `log_error_services` 中命名的组件顺序很重要，特别是与过滤器和接收器的相对顺序有关。考虑这个 `log_error_services` 值：

    ```sql
    log_filter_internal; log_sink_1; log_sink_2
    ```

    在这种情况下，日志事件经过内置过滤器，然后传递到第一个接收器，再传递到第二个接收器。两个接收器都会接收到经过过滤的日志事件。

    将其与此 `log_error_services` 值进行比较：

    ```sql
    log_sink_1; log_filter_internal; log_sink_2
    ```

    在这种情况下，日志事件经过第一个接收器，然后经过内置过滤器，再传递到第二个接收器。第一个接收器接收未经过滤的事件。第二个接收器接收经过过滤的事件。如果您希望一个日志包含所有日志事件的消息，另一个日志仅包含部分日志事件的消息，您可以以这种方式配置错误日志记录。

##### 错误日志配置方法

错误日志配置涉及根据需要加载和启用错误日志组件，并执行特定于组件的配置。

有两种错误日志配置方法，*隐式* 和 *显式*。建议选择一种配置方法并专门使用。同时使用两种方法可能会导致启动时出现警告。有关更多信息，请参阅 故障排除配置问题。

+   *隐式错误日志配置*（MySQL 8.0.30 中引入）

    此配置方法加载并启用由 `log_error_services` 变量定义的日志组件。在启动时，尚未加载的可加载组件会在`InnoDB`存储引擎完全可用之前隐式加载。此配置方法具有以下优点：

    +   日志组件在启动序列的早期加载，早于`InnoDB`存储引擎，使得记录的信息更早可用。

    +   它避免了在启动过程中发生故障时丢失缓冲的日志信息。

    +   使用 `INSTALL COMPONENT` 安装错误日志组件并不是必需的，简化了错误日志配置。

    要使用这种方法，请参阅 隐式错误日志配置。

+   *显式错误日志配置*

    注意

    此配置方法支持向后兼容。推荐使用 MySQL 8.0.30 中引入的隐式配置方法。

    这种配置方法需要使用`INSTALL COMPONENT`加载错误日志组件，然后配置`log_error_services`以启用日志组件。`INSTALL COMPONENT`将组件添加到`mysql.component`表（一个`InnoDB`表），启动时要加载的组件从该表中读取，该表只有在`InnoDB`初始化后才能访问。

    在`InnoDB`存储引擎初始化期间，启动序列期间缓冲记录的信息，这有时会因为`InnoDB`启动序列期间发生的恢复和数据字典升级等操作而延长。

    要使用此方法，请参阅显式错误日志配置。

##### 隐式错误日志配置

本过程描述了如何使用`log_error_services`隐式加载和启用错误日志组件。有关错误日志配置方法的讨论，请参见错误日志配置方法。

隐式加载和启用错误日志组件：

1.  在`log_error_services`值中列出错误日志组件。

    要在服务器启动时加载和启用错误日志组件，请在选项文件中设置`log_error_services`。以下示例配置了使用 JSON 日志接收器（`log_sink_json`）以及内置日志过滤器和接收器（`log_filter_internal`，`log_sink_internal`）。

    ```sql
    [mysqld]
    log_error_services='log_filter_internal; log_sink_internal; log_sink_json'
    ```

    注意

    要使用 JSON 日志接收器（`log_sink_syseventlog`）而不是默认接收器（`log_sink_internal`），您应该将`log_sink_internal`替换为`log_sink_json`。

    要立即加载和启用组件并用于后续重新启动，请使用`SET PERSIST`设置`log_error_services`：

    ```sql
    SET PERSIST log_error_services = 'log_filter_internal; log_sink_internal; log_sink_json';
    ```

1.  如果错误日志组件公开任何必须设置的系统变量以使组件初始化成功，请为这些变量分配适当的值。您可以在选项文件中设置这些变量，也可以使用`SET PERSIST`。

    重要

    在实现隐式配置时，首先设置`log_error_services`以加载组件并公开其系统变量，然后设置组件系统变量。无论是在命令行、选项文件还是使用`SET PERSIST`进行变量赋值，都需要按照此配置顺序进行。

要禁用日志组件，请从`log_error_services`值中移除它。同时移除您定义的任何相关组件变量设置。

注意

使用`log_error_services`隐式加载日志组件不会影响`mysql.component`表。它不会将组件添加到`mysql.component`表中，也不会从`mysql.component`表中删除以前使用`INSTALL COMPONENT`安装的组件。

##### 明确的错误日志配置

本过程描述了如何通过使用`INSTALL COMPONENT`加载组件，然后通过`log_error_services`启用错误日志组件。有关错误日志配置方法的讨论，请参见错误日志配置方法。

明确加载和启用错误日志组件：

1.  使用`INSTALL COMPONENT`加载组件（除非它是内置的或已加载）。例如，要加载 JSON 日志接收器，请执行以下语句：

    ```sql
    INSTALL COMPONENT 'file://component_log_sink_json';
    ```

    使用`INSTALL COMPONENT`加载组件会将其注册到`mysql.component`系统表中，以便服务器在`InnoDB`初始化后自动加载它以供后续启动使用。

    使用`INSTALL COMPONENT`加载日志组件时使用的 URN 是组件名称前缀为`file://component_`。例如，对于`log_sink_json`组件，相应的 URN 是`file://component_log_sink_json`。有关错误日志组件 URN，请参见 Section 7.5.3, “Error Log Components”。

1.  如果错误日志组件公开了必须设置的系统变量以确保组件初始化成功，为这些变量分配适当的值。您可以在选项文件中设置这些变量，也可以使用`SET PERSIST`。

1.  通过在`log_error_services`值中列出组件来启用该组件。

    重要

    从 MySQL 8.0.30 开始，当使用`INSTALL COMPONENT`显式加载日志组件时，请勿在选项文件中持久化或设置`log_error_services`，该选项文件在启动时隐式加载日志组件。而是使用`SET GLOBAL`语句在运行时启用日志组件。

    以下示例配置了使用 JSON 日志接收器（`log_sink_json`）以及内置日志过滤器和接收器（`log_filter_internal`，`log_sink_internal`）。

    ```sql
    SET GLOBAL log_error_services = 'log_filter_internal; log_sink_internal; log_sink_json';
    ```

    注意

    要使用 JSON 日志接收器（`log_sink_syseventlog`）而不是默认接收器（`log_sink_internal`），您应该将`log_sink_internal`替换为`log_sink_json`。

要禁用日志组件，请从`log_error_services`值中删除它。然后，如果组件是可加载的，并且您还想卸载它，请使用`UNINSTALL COMPONENT`。还要删除您定义的任何相关组件变量设置。

尝试使用`UNINSTALL COMPONENT`卸载仍在`log_error_services`值中命名的可加载组件会产生错误。

##### 更改错误日志配置方法

如果您之前使用`INSTALL COMPONENT`显式加载了错误日志组件，并且希望切换到隐式配置，如隐式错误日志配置中所述，建议执行以下步骤：

1.  将`log_error_services`设置回默认配置。

    ```sql
    SET GLOBAL log_error_services = 'log_filter_internal,log_sink_internal';
    ```

1.  使用`UNINSTALL COMPONENT`卸载您之前安装的任何可加载日志组件。例如，如果之前安装了 JSON 日志接收器，请按照以下方式��载它：

    ```sql
    UNINSTALL COMPONENT 'file://component_log_sink_json';
    ```

1.  删除已卸载组件的任何组件变量设置。例如，如果在选项文件中设置了组件变量，请从选项文件中删除这些设置。如果使用`SET PERSIST`设置了组件变量，请使用`RESET PERSIST`来清除这些设置。

1.  按照隐式错误日志配置中的步骤重新实施您的配置。

如果需要从隐式配置恢复到显式配置，请执行以下步骤：

1.  将`log_error_services`设置回其默认配置以卸载隐式加载的日志组件。

    ```sql
    SET GLOBAL log_error_services = 'log_filter_internal,log_sink_internal';
    ```

1.  删除与已卸载组件相关的任何组件变量设置。例如，如果在选项文件中设置了组件变量，请从选项文件中删除这些设置。如果使用`SET PERSIST`设置了组件变量，请使用`RESET PERSIST`清除这些设置。

1.  重新启动服务器以卸载隐式加载的日志组件。

1.  按照显式错误日志配置中的步骤重新实施您的配置。

##### 故障排除配置问题

从 MySQL 8.0.30 开始，在启动时加载在`log_error_services`值中列出的日志组件会在 MySQL 服务器启动序列的早期隐式加载。如果以前使用`INSTALL COMPONENT`加载了日志组件，则服务器会在启动序列的后期尝试重新加载该组件，从而产生以下警告：

```sql
Cannot load component from specified URN: 'file://component_*component_name*'
```

您可以在错误日志中检查此警告，或通过以下查询查询性能模式`error_log`表来查询此警告：

```sql
SELECT error_code, data
  FROM performance_schema.error_log
 WHERE data LIKE "%'file://component_%"
   AND error_code="MY-013129" AND data LIKE "%MY-003529%";
```

要避免此警告，请按照更改错误日志配置方法中的说明调整您的错误日志配置。应使用隐式或显式错误日志配置，但不要同时使用两者。

当尝试显式加载在启动时隐式加载的组件时会出现类似错误。例如，如果`log_error_services`列出了 JSON 日志接收器组件，则该组件会在启动时隐式加载。尝试稍后显式加载相同组件会返回此错误：

```sql
mysql> INSTALL COMPONENT 'file://component_log_sink_json';
ERROR 3529 (HY000): Cannot load component from specified URN: 'file://component_log_sink_json'.
```

##### 配置多个日志接收器

可以配置多个日志接收器，从而可以将输出发送到多个目的地。要在默认接收器之外启用 JSON 日志接收器（而不是替代），请将`log_error_services`值设置如下：

```sql
SET GLOBAL log_error_services = 'log_filter_internal; log_sink_internal; log_sink_json';
```

要恢复仅使用默认接收器并卸载系统日志接收器，请执行以下语句：

```sql
SET GLOBAL log_error_services = 'log_filter_internal; log_sink_internal;
UNINSTALL COMPONENT 'file://component_log_sink_json';
```

##### 日志接收器性能模式支持

如果启用了记录组件包含支持性能模式的接收器，那么写入错误日志的事件也会写入性能模式的`error_log`表中。这样可以使用 SQL 查询来检查错误日志内容。目前，传统格式的`log_sink_internal`和 JSON 格式的`log_sink_json`接收器支持此功能。请参阅 Section 29.12.21.2, “错误日志表”。
