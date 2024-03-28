# 10.12 优化 MySQL 服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-server.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-server.html)

10.12.1 优化磁盘 I/O

10.12.2 使用符号链接

10.12.3 优化内存使用

本节讨论了数据库服务器的优化技术，主要涉及系统配置而不是调整 SQL 语句。本节中的信息适用于希望确保管理的服务器性能和可伸缩性的数据库管理员；为构建包括设置数据库的安装脚本的开发人员；以及自行运行 MySQL 进行开发、测试等的人，他们希望最大化自己的生产力。
