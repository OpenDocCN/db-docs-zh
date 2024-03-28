# 2.8.1 源码安装方法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/source-installation-methods.html`](https://dev.mysql.com/doc/refman/8.0/en/source-installation-methods.html)

有两种方法可以从源码安装 MySQL：

+   使用标准 MySQL 源码发行版。要获取标准发行版，请参阅第 2.1.3 节，“如何获取 MySQL”。有关从标准发行版构建的说明，请参阅第 2.8.4 节，“使用标准源码发行版安装 MySQL”。

    标准发行版可作为压缩的**tar**文件、Zip 存档或 RPM 软件包提供。发行文件的名称形式为`mysql-*`VERSION`*.tar.gz`、`mysql-*`VERSION`*.zip`或`mysql-*`VERSION`*.rpm`，其中*`VERSION`*是一个类似`8.0.36`的数字。源码发行版的文件名可以通过源码发行版名称是通用的且不包含平台名称来与预编译二进制发行版的文件名区分开，而二进制发行版的文件名包含指示发行版面向的系统类型的平台名称（例如`pc-linux-i686`或`winx64`）。

+   使用 MySQL 开发树。有关从开发树构建的信息，请参阅第 2.8.5 节，“使用开发源树安装 MySQL”。
