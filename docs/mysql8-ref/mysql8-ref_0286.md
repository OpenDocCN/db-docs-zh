# 7.5.5 调度程序组件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/scheduler-component.html`](https://dev.mysql.com/doc/refman/8.0/en/scheduler-component.html)

注意

`scheduler`组件包含在 MySQL 企业版中，这是一款商业产品。要了解更多关于商业产品的信息，请参见[`www.mysql.com/products/`](https://www.mysql.com/products/)。

截至 MySQL 8.0.34，`scheduler`组件提供了`mysql_scheduler`服务的实现，使应用程序、组件或插件可以每隔*N*秒配置、运行和取消配置任务。例如，`audit_log`服务器插件在初始化时调用`scheduler`组件，并配置定期刷新其内存缓存（参见启用审计日志刷新任务）。

+   目的：实现`component_scheduler.enabled`系统变量，控制调度程序是否正在执行任务。在启动时，`scheduler`组件注册`performance_schema.component_scheduler_tasks`表，列出当前计划任务以及关于每个任务的一些运行时数据。

+   URN：`file://component_sheduler`

安装说明，请参见第 7.5.1 节，“安装和卸载组件”。

`scheduler`组件使用以下元素实现服务：

+   一个按照下次运行时间（升序）排序的注册的非活动计划任务的优先级队列。

+   一个注册的活动任务列表。

+   一个后台线程：

    +   如果没有任务或者顶部任务需要更多时间来运行，则休眠。它会定期唤醒以检查是否到达结束时间。

    +   编译需要运行的任务列表，将其从非活动队列移动到活动队列，并逐个执行每个任务。

    +   执行任务列表后，将任务从活动列表中移除，添加到非活动列表，并计算它们下次需要运行的时间。

当调用者调用`mysql_scheduler.create()`服务时，它会创建一个新的计划任务实例添加到队列中，这会向后台线程的信号量发送信号。将新任务的句柄返回给调用者。调用代码应保留此句柄和调度服务的引用，直到调用`mysql_scheduler.destroy()`服务后。当调用者调用`destroy()`并传入从`create()`接收的句柄时，服务会等待任务变为非活动状态（如果正在运行），然后将其从非活动队列中移除。

组件服务调用每个应用程序提供的回调（函数指针）到同一调度线程中，依次按照每个需要运行的时间的顺序。

希望将调度队列功能整合到应用程序、组件或插件中的开发人员应查阅 MySQL 源代码分发中的 `mysql_scheduler.h` 文件。
