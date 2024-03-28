# 7.5.3 错误日志组件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-components.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-components.html)

本节描述了各个错误日志组件的特性。有关配置错误日志的一般信息，请参见 Section 7.4.2, “The Error Log”。

日志组件可以是过滤器或接收器：

+   过滤器处理日志事件，以添加、删除或修改事件字段，或完全删除事件。处理后的事件传递到已启用组件列表中的下一个日志组件。

+   接收器是日志事件的目的地（写入器）。通常，接收器将日志事件处理成具有特定格式的日志消息，并将这些消息写入其关联的输出，如文件或系统日志。接收器还可以写入到性能模式`error_log`表；参见 Section 29.12.21.2, “The error_log Table”。事件未经修改地传递到已启用组件列表中的下一个日志组件（即，尽管接收器格式化事件以生成输出消息，但它不会在内部传递给下一个组件时修改事件）。

`log_error_services`系统变量值列出了已启用的日志组件。未在列表中命名的组件已禁用。从 MySQL 8.0.30 开始，如果未加载错误日志组件，则`log_error_services`也会隐式加载它们。有关更多信息，请参见 Section 7.4.2.1, “Error Log Configuration”。

以下各节描述了按组件类型分组的各个日志组件：

+   过滤器错误日志组件

+   接收器错误日志组件

组件描述包括以下类型的信息：

+   组件名称和预期目的。

+   组件是内置的还是必须加载。对于可加载组件，描述指定了在使用`INSTALL COMPONENT`和`UNINSTALL COMPONENT`语句显式加载或卸载组件时要使用的 URN。隐式加载错误日志组件只需要组件名称。有关更多信息，请参见 Section 7.4.2.1, “Error Log Configuration”。

+   组件是否可以在`log_error_services`值中列出多次。

+   对于一个接收组件，组件写入输出的目的地。

+   对于一个接收组件，它是否支持与性能模式`error_log`表的接口。

#### 过滤错误日志组件

错误日志过滤组件实现对错误日志事件的过滤。如果没有启用过滤组件，则不会进行过滤。

任何启用的过滤组件仅影响稍后在`log_error_services`值中列出的组件的日志事件。特别是，对于在`log_error_services`中较早列出的任何日志接收组件，不会发生任何日志事件过滤。

##### log_filter_internal 组件

+   目的：实现基于日志事件优先级和错误代码的过滤，结合`log_error_verbosity`和`log_error_suppression_list`系统变量。参见第 7.4.2.5 节“基于优先级的错误日志过滤（log_filter_internal）”。

+   URN：此组件是内置的，无需加载。

+   允许多次使用：否。

如果`log_filter_internal`被禁用，`log_error_verbosity`和`log_error_suppression_list`将不起作用。

##### log_filter_dragnet 组件

+   目的：实现根据`dragnet.log_error_filter_rules`系统变量设置定义的规则进行过滤。参见第 7.4.2.6 节“基于规则的错误日志过滤（log_filter_dragnet）”。

+   URN：`file://component_log_filter_dragnet`

+   允许多次使用：否。

#### 接收错误日志组件

错误日志接收组件是实现错误日志输出的写入器。如果没有启用接收组件，则不会发生日志输出。

一些接收组件描述涉及默认的错误日志目的地。这是控制台或文件，并由`log_error`系统变量的值表示，如第 7.4.2.2 节“默认错误日志目的地配置”中所述确定。

##### log_sink_internal 组件

+   目的：实现传统的错误日志消息输出格式。

+   URN：此组件是内置的，无需加载。

+   允许多次使用：否。

+   输出目的地：写入默认的错误日志目的地。

+   性能模式支持：写入`error_log`表。提供一个解析器用于读取以前服务器实例创建的错误日志文件。

##### JSON 格式日志汇聚组件

+   目的：实现 JSON 格式的错误日志记录。参见第 7.4.2.7 节，“JSON 格式错误日志记录”。

+   URN：`file://component_log_sink_json`

+   多次使用允许：允许。

+   输出目的地：该汇聚根据默认错误日志目的地确定其输出目的地，该目的地由`log_error`系统变量给出：

    +   如果`log_error`指定了一个文件，汇聚将基于该文件名进行输出文件命名，加上一个以`.*`NN`*.json`为后缀的编号，其中*`NN`*从 00 开始。例如，如果`log_error`是*`file_name`*，那么在`log_error_services`值中连续命名的`log_sink_json`将写入`*`file_name`*.00.json`，`*`file_name`*.01.json`等。

    +   如果`log_error`是`stderr`，则汇聚将写入控制台。如果在`log_error_services`值中多次命名`log_sink_json`，它们都将写入控制台，这可能没有用处。

+   性能模式支持：写入`error_log`表。提供一个解析器用于读取以前服务器实例创建的错误日志文件。

##### 日志汇聚系统事件日志组件

+   目的：实现将错误日志记录到系统日志。这是 Windows 上的事件日志，以及 Unix 和类 Unix 系统上的`syslog`。参见第 7.4.2.8 节，“将错误日志记录到系统日志”。

+   URN：`file://component_log_sink_syseventlog`

+   多次使用允许：不允许。

+   输出目的地：写入系统日志。不使用默认错误日志目的地。

+   性能模式支持：不写入`error_log`表。不提供解析器用于读取以前服务器实例创建的错误日志文件。

##### 日志汇聚测试组件

+   目的：用于编写测试用例的内部使用，不用于生产环境。

+   URN：`file://component_log_sink_test`

由于`log_sink_test`是用于内部使用，因此未指定诸如是否允许多次使用和输出目的地等汇聚属性。因此，其行为可能随时发生变化。
