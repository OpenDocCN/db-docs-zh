# 8.5.2 MySQL 企业数据脱敏和去标识化组件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-components.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-components.html)

8.5.2.1 MySQL 企业数据脱敏和去标识化组件安装

8.5.2.2 使用 MySQL 企业数据脱敏和去标识化组件

8.5.2.3 MySQL 企业数据脱敏和去标识化组件功能参考

8.5.2.4 MySQL 企业数据脱敏和去标识化组件功能描述

MySQL 企业数据脱敏和去标识化实现了以下元素：

+   `mysql`系统数据库中的一张表，用于持久存储字典和术语。

+   一个名为`component_masking`的组件，实现脱敏功能并将其作为服务接口提供给开发人员。

    希望使用`component_masking`相同服务功能的开发人员应查阅 MySQL 源代码分发中的`internal\components\masking\component_masking.h`文件或 https://dev.mysql.com/doc/dev/mysql-server/latest。

+   一个名为`component_masking_functions`的组件，提供可加载函数。

    一组可加载函数为执行脱敏和去标识化操作提供了 SQL 级 API。其中一些函数需要`MASKING_DICTIONARIES_ADMIN`动态权限。
