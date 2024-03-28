# 8.5.3 MySQL 企业数据脱敏和去标识化插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-plugin.html)

8.5.3.1 MySQL 企业数据脱敏和去标识化插件安装

8.5.3.2 使用 MySQL 企业数据脱敏和去标识化插件

8.5.3.3 MySQL 企业数据脱敏和去标识化插件功能参考

8.5.3.4 MySQL 企业数据脱敏和去标识化插件功能描述

MySQL 企业数据脱敏和去标识化基于一个实现这些元素的插件库：

+   一个名为`data_masking`的服务器端插件。

+   一组可加载的函数提供了一个 SQL 级别的 API，用于执行脱敏和去标识化操作。其中一些函数需要`SUPER`权限。
