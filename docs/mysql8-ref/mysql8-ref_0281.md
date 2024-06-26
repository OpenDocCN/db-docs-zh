# 7.5 MySQL 组件

> 译文：[`dev.mysql.com/doc/refman/8.0/en/components.html`](https://dev.mysql.com/doc/refman/8.0/en/components.html)

7.5.1 安装和卸载组件

7.5.2 获取组件信息

7.5.3 错误日志组件

7.5.4 查询属性组件

7.5.5 调度器组件

MySQL Server 包括一个基于组件的基础架构，用于扩展服务器功能。组件提供对服务器和其他组件可用的服务。（在服务使用方面，服务器是一个组件，与其他组件相等。）组件之间仅通过它们提供的服务进行交互。

MySQL 发行版包含几个实现服务器扩展的组件：

+   用于配置错误日志的组件。参见第 7.4.2 节，“错误日志”和第 7.5.3 节，“错误日志组件”。

+   一个用于检查密码的组件。参见第 8.4.3 节，“密码验证组件”。

+   Keyring 组件为敏感信息提供安全存储。参见第 8.4.4 节，“MySQL Keyring”。

+   一个使应用程序能够向审计日志添加自己的消息事件的组件。参见第 8.4.6 节，“审计消息组件”。

+   实现用于访问查询属性的可加载函数的组件。参见第 11.6 节，“查询属性”。

+   用于调度主动执行任务的组件。参见第 7.5.5 节，“调度器组件”。

当组件安装时，由组件实现的系统和状态变量将被公开，并具有以组件特定前缀开头的名称。例如，`log_filter_dragnet`错误日志过滤组件实现了一个名为`log_error_filter_rules`的系统变量，其完整名称为`dragnet.log_error_filter_rules`。要引用此变量，请使用完整名称。

以下部分描述了如何安装和卸载组件，以及在运行时如何确定已安装的组件并获取有关它们的信息。

有关组件的内部实现信息，请参阅 MySQL Server Doxygen 文档，网址为`dev.mysql.com/doc/index-other.html`。例如，如果您打算编写自己的组件，了解这些信息对于理解组件的工作原理非常重要。
