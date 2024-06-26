# 10.13 性能测量（基准测试）

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimize-benchmarking.html`](https://dev.mysql.com/doc/refman/8.0/en/optimize-benchmarking.html)

10.13.1 测量表达式和函数的速度

10.13.2 使用自定义基准

10.13.3 使用 performance_schema 测量性能

为了衡量性能，请考虑以下因素：

+   无论您是在安静系统上测量单个操作的速度，还是在一段时间内测试一组操作（“工作负载”）的工作方式。通过简单测试，通常测试改变一个方面（配置设置、表上的索引集合、查询中的 SQL 子句）如何影响性能。基准测试通常是长时间运行和复杂的性能测试，结果可能决定高级选择，如硬件和存储配置，或者何时升级到新的 MySQL 版本。

+   对于基准测试，有时必须模拟繁重的数据库工作负载以获得准确的图片。

+   性能可能会因为许多不同因素而变化，几个百分点的差异可能不是决定性的胜利。当在不同环境中进行测试时，结果可能会相反。

+   某些 MySQL 功能根据工作负载是否有助于性能。为了全面性，始终测试开启和关闭这些功能的性能。对于每种工作负载尝试的最重要功能是`InnoDB`表的自适应哈希索引。

本节从单个开发人员可以执行的简单直接的测量技术开始，逐渐发展到需要额外专业知识来执行和解释结果的更复杂的技术。
