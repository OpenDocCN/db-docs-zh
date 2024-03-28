# 2.1.6 编译器特定的构建特性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/compiler-characteristics.html`](https://dev.mysql.com/doc/refman/8.0/en/compiler-characteristics.html)

在某些情况下，用于构建 MySQL 的编译器会影响可用的功能。本节中的注释适用于由 Oracle 公司提供的二进制发行版，或者您从源代码自行编译的情况。

****icc**（英特尔 C++ 编译器）构建**

使用**icc**构建的服务器具有以下特点：

+   不包括 SSL 支持。
