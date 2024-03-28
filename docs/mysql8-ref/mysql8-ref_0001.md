# 前言和法律声明

> 原文：[`dev.mysql.com/doc/refman/8.0/en/preface.html`](https://dev.mysql.com/doc/refman/8.0/en/preface.html)

这是 MySQL 数据库系统版本 8.0 的参考手册，直至 8.0.36 版本。MySQL 8.0 的次要版本之间的差异在本文中以发布编号（8.0.*`x`*）的形式进行标注。有关许可信息，请参阅法律声明。

由于 MySQL 8.0 与旧版本之间存在许多功能和其他差异，本手册不适用于旧版本的 MySQL 软件。如果您使用的是 MySQL 软件的早期版本，请参考相应的手册。例如，*MySQL 5.7 参考手册*涵盖了 MySQL 软件 5.7 系列的发布。

**许可信息—MySQL 8.0. ** 本产品可能包含根据许可使用的第三方软件。如果您使用的是*商业*版的 MySQL 8.0，请参阅[MySQL 8.0 商业版许可信息用户手册](https://downloads.mysql.com/docs/licenses/mysqld-8.0-com-en.pdf)获取许可信息，包括与可能包含在此商业版中的第三方软件相关的许可信息。如果您使用的是*社区*版的 MySQL 8.0，请参阅[MySQL 8.0 社区版许可信息用户手册](https://downloads.mysql.com/docs/licenses/mysqld-8.0-gpl-en.pdf)获取许可信息，包括与可能包含在此社区版中的第三方软件相关的许可信息。

**许可信息—MySQL NDB Cluster 8.0. ** 如果您使用的是*商业*版的 MySQL NDB Cluster 8.0，请参阅[MySQL NDB Cluster 8.0 商业版许可信息用户手册](https://downloads.mysql.com/docs/licenses/cluster-8.0-com-en.pdf)获取许可信息，包括与可能包含在此商业版中的第三方软件相关的许可信息。如果您使用的是*社区*版的 MySQL NDB Cluster 8.0，请参阅[MySQL NDB Cluster 8.0 社区版许可信息用户手册](https://downloads.mysql.com/docs/licenses/cluster-8.0-gpl-en.pdf)获取许可信息，包括与可能包含在此社区版中的第三方软件相关的许可信息。

## 法律声明

版权 © 1997 年至 2024 年，Oracle 及/或其关联公司。

**许可限制**

本软件及相关文档受许可协议保护，包含对使用和披露的限制，并受知识产权法保护。除非您的许可协议明确允许或法律允许，否则您不得使用、复制、翻译、广播、修改、许可、传输、分发、展示、执行、发布或显示任何部分，以任何形式或任何方式。除非法律要求用于互操作性，否则禁止对本软件进行逆向工程、反汇编或解编译。

**担保免责声明**

此处包含的信息可能会更改，不保证无错误。如果发现任何错误，请以书面形式向我们报告。

**受限权利声明**

如果这是交付给美国政府或代表美国政府许可的软件、软件文档、数据（如联邦采购条例中定义的）或相关文档，则以下通知适用：

美国政府最终用户：交付给或由美国政府最终用户访问的 Oracle 程序（包括任何操作系统、集成软件、安装在交付硬件上的任何程序以及对这些程序的修改）和 Oracle 计算机文档或其他 Oracle 数据均为“商业计算机软件”，“商业计算机软件文档”或“有限权利数据”，根据适用的联邦采购条例和机构特定的补充规定。因此，对 i）Oracle 程序（包括任何操作系统、集成软件、安装在交付硬件上的任何程序以及对这些程序的修改）、ii）Oracle 计算机文档和/或 iii）其他 Oracle 数据的使用、复制、复制、发布、显示、披露、修改、准备衍生作品和/或适应性，受限于适用合同中包含的许可中指定的权利和限制。美国政府使用 Oracle 云服务的条款由适用于此类服务的合同定义。未授予美国政府其他权利。

**危险应用通知**

此软件或硬件是为各种信息管理应用程序的一般使用而开发的。它不是为任何本质上危险的应用程序开发或拟定，包括可能造成个人伤害风险的应用程序。如果您在危险应用中使用此软件或硬件，则应负责采取所有适当的故障安全、备份、冗余和其他措施以确保其安全使用。Oracle Corporation 及其关联公司对在危险应用中使用此软件或硬件造成的任何损害不承担任何责任。

**商标声明**

Oracle，Java，MySQL 和 NetSuite 是 Oracle 及/或其关联公司的注册商标。其他名称可能是其各自所有者的商标。

英特尔和英特尔标志是英特尔公司的商标或注册商标。所有 SPARC 商标均在许可下使用，并且是 SPARC International, Inc.的商标或注册商标。AMD，Epyc 和 AMD 标志是 Advanced Micro Devices 的商标或注册商标。UNIX 是 The Open Group 的注册商标。

**第三方内容、产品和服务免责声明**

此软件或硬件和文档可能提供对第三方内容、产品和服务的访问或信息。除非另有适用协议中规定，否则 Oracle 公司及其关联公司对第三方内容、产品和服务的所有类型的任何保证均不负责并明确否认。除非另有适用协议中规定，否则 Oracle 公司及其关联公司不会对您访问或使用第三方内容、产品或服务而造成的任何损失、成本或损害负责。

**使用此文档**

此文档不是根据 GPL 许可证分发的。使用此文档受以下条款约束：

您可以仅为个人使用创建此文档的打印副本。允许转换为其他格式，只要实际内容没有被修改或编辑。您不得以任何形式或媒体发布或分发此文档，除非您以类似于 Oracle 发布方式（即通过网站电子下载软件）或在 CD-ROM 或类似介质上分发文档，但前提是文档与软件一起在同一介质上分发。任何其他用途，如分发印刷副本或在其他出版物中全部或部分使用此文档，需要 Oracle 授权代表的事先书面同意。Oracle 及/或其关联公司保留未明示授予的此文档的任何权利。

## 文档无障碍功能

有关 Oracle 致力于无障碍功能的信息，请访问 Oracle 无障碍计划网站 [`www.oracle.com/pls/topic/lookup?ctx=acc&id=docacc`](http://www.oracle.com/pls/topic/lookup?ctx=acc&id=docacc)。

## 访问 Oracle 支持无障碍功能

已购买支持的 Oracle 客户可通过 My Oracle Support 访问电子支持。有关信息，请访问 [`www.oracle.com/pls/topic/lookup?ctx=acc&id=info`](http://www.oracle.com/pls/topic/lookup?ctx=acc&id=info) 或访问 `[`www.oracle.com/pls/topic/lookup?ctx=acc&id=trs`](http://www.oracle.com/pls/topic/lookup?ctx=acc&id=trs)`（听力障碍者）。
