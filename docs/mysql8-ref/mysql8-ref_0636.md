# 10.13.2 使用您自己的基准测试

> 原文：[`dev.mysql.com/doc/refman/8.0/en/custom-benchmarks.html`](https://dev.mysql.com/doc/refman/8.0/en/custom-benchmarks.html)

对应用程序和数据库进行基准测试，找出瓶颈所在。在解决一个瓶颈（或用“虚拟”模块替换它）之后，您可以继续识别下一个瓶颈。即使您当前的应用程序整体性能是可接受的，您也应该至少为每个瓶颈制定计划，并决定如何解决它，以防有一天您真的需要额外的性能。

一个免费的基准测试套件是开源数据库基准测试，可在[`osdb.sourceforge.net/`](http://osdb.sourceforge.net/)找到。

当系统负载非常重时，问题只会发生是非常常见的。我们曾经有许多客户在生产中遇到负载问题时联系我们。在大多数情况下，性能问题往往是由于基本数据库设计问题（例如，在高负载下表扫描效果不佳）或操作系统或库的问题。如果系统尚未投入生产，这些问题大多数情况下会更容易解决。

为避免出现此类问题，请在最坏的负载情况下对整个应用程序进行基准测试：

+   **mysqlslap**程序对于模拟多个客户端同时发出查询产生的高负载非常有帮助。请参阅 Section 6.5.8, “mysqlslap — A Load Emulation Client”。

+   您还可以尝试诸如 SysBench 和 DBT2 等基准测试软件包，可在[`launchpad.net/sysbench`](https://launchpad.net/sysbench)和[`osdldbt.sourceforge.net/#dbt2`](http://osdldbt.sourceforge.net/#dbt2)找到。

这些程序或软件包可能会使系统崩溃，因此请确保仅在开发系统上使用它们。
