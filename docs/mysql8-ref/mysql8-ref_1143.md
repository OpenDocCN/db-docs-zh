# 第十七章 InnoDB 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)

**目录**

17.1 InnoDB 简介

17.1.1 使用 InnoDB 表的好处

17.1.2 InnoDB 表的最佳实践

17.1.3 验证 InnoDB 是否为默认存储引擎

17.1.4 使用 InnoDB 进行测试和基准测试

17.2 InnoDB 和 ACID 模型

17.3 InnoDB 多版本

17.4 InnoDB 架构

17.5 InnoDB 内存结构

17.5.1 缓冲池

17.5.2 变更缓冲区

17.5.3 自适应哈希索引

17.5.4 日志缓冲区

17.6 InnoDB 磁盘结构

17.6.1 表

17.6.2 索引

17.6.3 表空间

17.6.4 双写缓冲区

17.6.5 重做日志

17.6.6 撤销日志

17.7 InnoDB 锁定和事务模型

17.7.1 InnoDB 锁定

17.7.2 InnoDB 事务模型

17.7.3 InnoDB 中由不同 SQL 语句设置的锁

17.7.4 幻影行

17.7.5 InnoDB 中的死锁

17.7.6 事务调度

17.8 InnoDB 配置

17.8.1 InnoDB 启动配置

17.8.2 为只读操作配置 InnoDB

17.8.3 InnoDB 缓冲池配置

17.8.4 为 InnoDB 配置线程并发性

17.8.5 配置后台 InnoDB I/O 线程的数量

17.8.6 在 Linux 上使用异步 I/O

17.8.7 配置 InnoDB I/O 容量

17.8.8 配置自旋锁轮询

17.8.9 清理配置

17.8.10 为 InnoDB 配置优化器统计信息

17.8.11 配置索引页的合并阈值

17.8.12 启用专用 MySQL 服务器的自动配置

17.9 InnoDB 表和页面压缩

17.9.1 InnoDB 表压缩

17.9.2 InnoDB 页面压缩

17.10 InnoDB 行格式

17.11 InnoDB 磁盘 I/O 和文件空间管理

17.11.1 InnoDB 磁盘 I/O

17.11.2 文件空间管理

17.11.3 InnoDB 检查点

17.11.4 表格碎片整理

17.11.5 使用 TRUNCATE TABLE 回收磁盘空间

17.12 InnoDB 和在线 DDL

17.12.1 在线 DDL 操作

17.12.2 在线 DDL 性能和并发性

17.12.3 在线 DDL 空间需求

17.12.4 在线 DDL 内存管理

17.12.5 配置在线 DDL 操作的并行线程

17.12.6 使用在线 DDL 简化 DDL 语句

17.12.7 在线 DDL 失败条件

17.12.8 在线 DDL 限制

17.13 InnoDB 数据静态加密

17.14 InnoDB 启动选项和系统变量

17.15 InnoDB INFORMATION_SCHEMA 表格

17.15.1 关于压缩的 InnoDB INFORMATION_SCHEMA 表格

17.15.2 InnoDB INFORMATION_SCHEMA 事务和锁信息

17.15.3 InnoDB INFORMATION_SCHEMA 模式对象表

17.15.4 InnoDB INFORMATION_SCHEMA 全文索引表格

17.15.5 InnoDB INFORMATION_SCHEMA 缓冲池表格

17.15.6 InnoDB INFORMATION_SCHEMA 指标表

17.15.7 InnoDB INFORMATION_SCHEMA 临时表信息表

17.15.8 从 INFORMATION_SCHEMA.FILES 检索 InnoDB 表空间元数据

17.16 InnoDB 与 MySQL 性能模式的集成

17.16.1 使用性能模式监视 InnoDB 表的 ALTER TABLE 进度

17.16.2 使用性能模式监视 InnoDB 互斥等待

17.17 InnoDB 监视器

17.17.1 InnoDB 监视器类型

17.17.2 启用 InnoDB 监视器

17.17.3 InnoDB 标准监视器和锁监视器输出

17.18 InnoDB 备份和恢复

17.18.1 InnoDB 备份

17.18.2 InnoDB 恢复

17.19 InnoDB 和 MySQL 复制

17.20 InnoDB memcached 插件

17.20.1 InnoDB memcached 插件的优势

17.20.2 InnoDB memcached 架构

17.20.3 设置 InnoDB memcached 插件

17.20.4 InnoDB memcached 多次获取和范围查询支持

17.20.5 InnoDB memcached 插件的安全考虑

17.20.6 为 InnoDB memcached 插件编写应用程序

17.20.7 InnoDB memcached 插件和复制

17.20.8 InnoDB memcached 插件内部

17.20.9 修复 InnoDB memcached 插件的故障

17.21 InnoDB 故障排除

17.21.1 修复 InnoDB I/O 问题

17.21.2 修复恢复失败

17.21.3 强制 InnoDB 恢复

17.21.4 修复 InnoDB 数据字典操作

17.21.5 InnoDB 错误处理

17.22 InnoDB 限制

17.23 InnoDB 限制和限制
