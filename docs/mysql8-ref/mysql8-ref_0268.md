> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-json.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-json.html)

#### 7.4.2.7 JSON 格式错误日志

本节描述了如何使用内置过滤器`log_filter_internal`和 JSON sink`log_sink_json`立即生效并在后续服务器启动时生效。有关配置错误日志的一般信息，请参阅 Section 7.4.2.1, “错误日志配置”。

要启用 JSON sink，请首先加载 sink 组件，然后修改`log_error_services`的值：

```sql
INSTALL COMPONENT 'file://component_log_sink_json';
SET PERSIST log_error_services = 'log_filter_internal; log_sink_json';
```

要在服务器启动时设置`log_error_services`生效，请使用 Section 7.4.2.1, “错误日志配置”中的说明。这些说明也适用于其他错误日志系统变量。

在`log_error_services`的值中可以多次命名`log_sink_json`。例如，要使用一个实例写入未经过滤的事件，使用另一个实例写入经过滤的事件，可以设置`log_error_services`如下：

```sql
SET PERSIST log_error_services = 'log_sink_json; log_filter_internal; log_sink_json';
```

JSON sink 根据默认错误日志目的地确定其输出目的地，该目的地由`log_error`系统变量给出。如果`log_error`命名一个文件，则 JSON sink 基于该文件名进行输出文件命名，加上一个编号为`.*`NN`*.json`的后缀，其中*`NN`*从 00 开始。例如，如果`log_error`为*`file_name`*，则在`log_error_services`的值中连续的`log_sink_json`实例将写入`*`file_name`*.00.json`，`*`file_name`*.01.json`等。

如果`log_error`为`stderr`，则 JSON sink 写入控制台。如果在`log_error_services`的值中多次命名`log_sink_json`，它们都将写入控制台，这可能没有用处。
