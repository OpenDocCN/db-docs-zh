# 第一章 一般信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/introduction.html`](https://dev.mysql.com/doc/refman/8.0/en/introduction.html)

**目录**

1.1 关于本手册

1.2 MySQL 数据库管理系统概述

1.2.1 什么是 MySQL？

1.2.2 MySQL 的主要特性

1.2.3 MySQL 的历史

1.3 MySQL 8.0 中的新功能

1.4 MySQL 8.0 中添加、弃用或移除的服务器和状态变量和选项

1.5 如何报告错误或问题

1.6 MySQL 的标准合规性

1.6.1 MySQL 对标准 SQL 的扩展

1.6.2 MySQL 与标准 SQL 的不同之处

1.6.3 MySQL 如何处理约束

MySQL 软件提供了一个非常快速、多线程、多用户和强大的 SQL（结构化查询语言）数据库服务器。MySQL 服务器旨在用于关键任务、高负载生产系统，以及嵌入到大规模部署软件中。Oracle 是 Oracle 公司及/或其关联公司的注册商标。MySQL 是 Oracle 公司及/或其关联公司的商标，未经 Oracle 明确书面授权，客户不得使用。其他名称可能是其各自所有者的商标。

MySQL 软件具有双重许可。用户可以选择根据 GNU 通用公共许可证的条款将 MySQL 软件用作开源产品，或者可以从 Oracle 购买标准商业许可证。有关我们许可政策的更多信息，请参见[`www.mysql.com/company/legal/licensing/`](http://www.mysql.com/company/legal/licensing/)。

以下列表描述了本手册中一些特别感兴趣的部分：

+   有关 MySQL 数据库服务器功能的讨论，请参见 第 1.2.2 节，“MySQL 的主要特性”。

+   对于 MySQL 新功能的概述，请参见 第 1.3 节，“MySQL 8.0 中的新功能”。有关每个版本的更改信息，请参阅发布说明。

+   有关安装说明，请参见 第二章，“*安装 MySQL*”。有关升级 MySQL 的信息，请参见 第三章，“*升级 MySQL*”。

+   有关 MySQL 数据库服务器的教程介绍，请参见 第五章，“*教程*”。

+   有关配置和管理 MySQL 服务器的信息，请参见 第七章，“*MySQL 服务器管理*”。

+   有关 MySQL 安全性的信息，请参见 第八章，“*安全*”。

+   有关设置复制服务器的信息，请参阅 第十九章，*复制*。

+   有关 MySQL Enterprise，即具有高级功能和管理工具的商业 MySQL 发行版的信息，请参阅 第三十二章，*MySQL Enterprise Edition*。

+   有关 MySQL 数据库服务器及其功能的一些常见问题的答案，请参阅 附录 A，*MySQL 8.0 常见问题*。

+   要了解新功能和错误修复的历史，请参阅 发布说明。

重要提示

要报告问题或错误，请使用 第 1.5 节，“如何报告错误或问题” 中的说明。如果您在 MySQL Server 中发现安全漏洞，请立即通过电子邮件发送消息至 `<secalert_us@oracle.com>`。例外：支持客户应将所有问题，包括安全漏洞，报告给 Oracle 支持。
