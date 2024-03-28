# 8.5.1 数据遮蔽组件与数据遮蔽插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-masking-components-vs-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/data-masking-components-vs-plugin.html)

在 8.0.33 之前，MySQL 使用服务器端插件启用了遮蔽和去识别功能，但在 MySQL 8.0.33 中转而使用了组件基础架构。以下表格简要比较了 MySQL 企业数据遮蔽和去识别组件与插件库，以提供它们之间的区别概述。它可能帮助您从插件过渡到组件。

注意

只能同时启用数据遮蔽组件或插件。同时启用组件和插件不受支持，结果可能不如预期。

**表 8.45 数据遮蔽组件与插件元素的比较**

| 类别 | 组件 | 插件 |
| --- | --- | --- |
| 接口 | 服务函数、可加载函数 | 可加载函数 |
| 支持多字节字符集 | 是，用于通用遮蔽功能 | 否 |
| 通用遮蔽功能 | `mask_inner()`, `mask_outer()` | `mask_inner()`, `mask_outer()` |
| 特定类型的遮蔽 | PAN、SSN、IBAN、UUID、Canada SIN、UK NIN | PAN、SSN |
| 随机生成，特定类型 | 电子邮件、美国电话、PAN、SSN、IBAN、UUID、Canada SIN、UK NIN | 电子邮件、美国电话、PAN、SSN |
| 从给定范围生成整数的随机数 | 是 | 是 |
| 持久化替换字典 | 数据库 | 文件 |
| 管理字典的权限 | 专用权限 | 文件 |
| 安装/卸载期间自动加载功能注册/注销 | 是 | 否 |
| 现有功能的增强 | `gen_rnd_email()` 函数添加更多参数 | N/A |
| 类别 | 组件 | 插件 |
