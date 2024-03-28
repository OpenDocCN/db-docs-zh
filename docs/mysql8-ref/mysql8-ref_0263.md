> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-destination-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-destination-configuration.html)

#### 7.4.2.2 默认错误日志目的地配置

本节描述了哪些服务器选项配置了默认错误日志目的地，可以是控制台或命名文件。它还指出了哪些日志接收组件将其自身的输出目的地基于默认目的地。

在本讨论中，“控制台”指的是`stderr`，标准错误输出。这是您的终端或控制台窗口，除非标准错误输出已重定向到其他目的地。

服务器对于确定默认错误日志目的地的选项在 Windows 和 Unix 系统上有些不同。请确保使用适合您平台的信息配置目的地。服务器解释默认错误日志目的地选项后，它将设置`log_error`系统变量以指示默认目的地，这会影响几个日志接收组件写入错误消息的位置。以下各节将讨论这些主题。

+   Windows 上的默认错误日志目的地

+   Unix 和类 Unix 系统上的默认错误日志目的地

+   默认错误日志目的地如何影响日志接收组件

##### Windows 上的默认错误日志目的地

在 Windows 上，**mysqld**使用`--log-error`，`--pid-file`和`--console`选项来确定默认错误日志目的地是控制台还是文件，以及如果是文件，则文件名：

+   如果提供了`--console`，默认目的地是控制台。(`--console`优先于`--log-error`如果两者都提供，则关于`--log-error`的以下项目不适用。)

+   如果未提供`--log-error`，或者提供了但没有指定文件名，则默认目的地是数据目录中名为`*`host_name`*.err`的文件，除非指定了`--pid-file`选项。在这种情况下，文件名是 PID 文件基本名称，带有`.err`后缀在数据目录中。

+   如果给定`--log-error`来命名一个文件，那么默认的目的地就是该文件（如果名称没有后缀，则添加`.err`后缀）。文件位置位于数据目录下，除非给定绝对路径名以指定不同位置。

如果默认的错误日志目的地是控制台，服务器会将`log_error`系统变量设置为`stderr`。否则，默认目的地是一个文件，服务器会将`log_error`设置为文件名。

##### Unix 和类 Unix 系统上的默认错误日志目的地

在 Unix 和类 Unix 系统上，**mysqld**使用`--log-error`选项来确定默认的错误日志目的地是控制台还是文件，以及文件名：

+   如果没有给定`--log-error`，默认的目的地是控制台。

+   如果给定`--log-error`而没有命名文件，那么默认的目的地是数据目录中名为`*`host_name`*.err`的文件。

+   如果给定`--log-error`来命名一个文件，那么默认的目的地就是该文件（如果名称没有后缀，则添加`.err`后缀）。文件位置位于数据目录下，除非给定绝对路径名以指定不同位置。

+   如果在`[mysqld]`、`[server]`或`[mysqld_safe]`部分的选项文件中给定`--log-error`，在使用**mysqld_safe**启动服务器的系统上，**mysqld_safe**会找到并使用该选项，并将其传递给**mysqld**。

注意

对于 Yum 或 APT 软件包安装来说，通常会在服务器配置文件中使用类似`log-error=/var/log/mysqld.log`的选项来配置错误日志文件位置在`/var/log`下。从选项中移除路径名会导致在数据目录中使用`*`host_name`*.err`文件。

如果默认的错误日志目的地是控制台，服务器会将`log_error`系统变量设置为`stderr`。否则，默认目的地是一个文件，服务器会将`log_error`设置为文件名。

##### 默认错误日志目的地如何影响日志输出

在服务器解释错误日志目的地配置选项后，它将`log_error`系统变量设置为指示默认错误日志目的地。日志输出端组件可以基于`log_error`值确定自己的输出目的地，或者独立于`log_error`确定它们的目的地。

如果`log_error`为`stderr`，默认错误日志目的地为控制台，基于默认目的地的日志输出端也会写入控制台：

+   `log_sink_internal`, `log_sink_json`, `log_sink_test`: 这些输出端写入控制台。即使对于可以多次启用的输出端（例如`log_sink_json`），所有实例也会写入控制台。

+   `log_sink_syseventlog`: 此输出端写入系统日志，不受`log_error`值的影响。

如果`log_error`不是`stderr`，默认错误日志目的地是一个文件，并且`log_error`指示文件名。基于默认目的地的日志输出端会根据该文件名确定输出文件命名。 （一个输出端可能会使用完全相同的名称，或者可能会使用某种变体。）假设`log_error`值为*`file_name`*。那么日志输出端使用以下方式的名称：

+   `log_sink_internal`, `log_sink_test`: 这些输出端写入*`file_name`*。

+   `log_sink_json`: 连续的此输出端实例命名为`log_error_services`值写入名为*`file_name`*加上编号为`.*`NN`*.json`后缀的文件：`*`file_name`*.00.json`，`*`file_name`*.01.json`等等。

+   `log_sink_syseventlog`: 此输出端写入系统日志，不受`log_error`值的影响。
