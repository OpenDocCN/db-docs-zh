# 第三十四章 MySQL 在 OCI 市场上

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-oci-marketplace.html)

**目录**

34.1 在 Oracle 云基础设施上部署 MySQL EE 的先决条件

34.2 在 Oracle 云基础设施上部署 MySQL EE

34.3 配置网络访问

34.4 连接

34.5 维护

本章描述了如何将 MySQL 企业版部署为 Oracle 云基础设施（OCI）市场应用程序。这是一种 BYOL 产品。

注意

有关 OCI 市场的更多信息，请参阅[市场概述](https://docs.cloud.oracle.com/iaas/Content/Marketplace/Concepts/marketoverview.htm)。

MySQL 企业版市场应用程序是一个运行 Oracle Linux 7.9 的 OCI 计算实例，带有 MySQL EE 8.0。部署图像上的 MySQL EE 安装类似于 RPM 安装，如第 2.5.4 节，“使用来自 Oracle 的 RPM 包在 Linux 上安装 MySQL”中所述。

有关 MySQL 企业版的更多信息，请参阅第三十二章，*MySQL 企业版*。

有关 MySQL 高级配置的更多信息，请参阅安全部署指南。

有关 Oracle Linux 7 的更多信息，请参阅[Oracle Linux 文档](https://docs.oracle.com/en/operating-systems/oracle-linux/7/)

本产品由用户管理，这意味着您负责升级和维护。
