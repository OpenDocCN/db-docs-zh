# 32.7 MySQL 企业数据脱敏和去标识化概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-data-masking.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-data-masking.html)

MySQL 企业版 5.7 及更高版本包括 MySQL 企业数据脱敏和去标识化，实现为包含插件和多个可加载函数的插件库。数据脱敏通过用替代值替换真实值来隐藏敏感信息。MySQL 企业数据脱敏和去标识化函数使得可以使用多种方法对现有数据进行脱敏，例如混淆（删除识别特征）、生成格式化的随机数据以及数据替换或替代。

有关更多信息，请参阅第 8.5 节，“MySQL 企业数据脱敏和去标识化”。
