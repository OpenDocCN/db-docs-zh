# 第六章 MySQL 程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/programs.html`](https://dev.mysql.com/doc/refman/8.0/en/programs.html)

**目录**

6.1 MySQL 程序概述

6.2 使用 MySQL 程序

6.2.1 调用 MySQL 程序

6.2.2 指定程序选项

6.2.3 连接到服务器的命令选项

6.2.4 使用命令选项连接到 MySQL 服务器

6.2.5 使用类似 URI 字符串或键值对连接到服务器

6.2.6 使用 DNS SRV 记录连接到服务器

6.2.7 连接传输协议

6.2.8 连接压缩控制

6.2.9 设置环境变量

6.3 服务器和服务器启动程序

6.3.1 mysqld — MySQL 服务器

6.3.2 mysqld_safe — MySQL 服务器启动脚本

6.3.3 mysql.server — MySQL 服务器启动脚本

6.3.4 mysqld_multi — 管理多个 MySQL 服务器

6.4 与安装相关的程序

6.4.1 comp_err — 编译 MySQL 错误消息文件

6.4.2 mysql_secure_installation — 改善 MySQL 安装安全性

6.4.3 mysql_ssl_rsa_setup — 创建 SSL/RSA 文件

6.4.4 mysql_tzinfo_to_sql — 加载时区表

6.4.5 mysql_upgrade — 检查和升级 MySQL 表

6.5 客户端程序

6.5.1 mysql — MySQL 命令行客户端

6.5.2 mysqladmin — 一个 MySQL 服务器管理程序

6.5.3 mysqlcheck — 一个表维护程序

6.5.4 mysqldump — 一个数据库备份程序

6.5.5 mysqlimport — 一个数据导入程序

6.5.6 mysqlpump — 一个数据库备份程序

6.5.7 mysqlshow — 显示数据库、表和列信息

6.5.8 mysqlslap — 一个负载仿真客户端

6.6 管理和实用程序

6.6.1 ibd2sdi — InnoDB 表空间 SDI 提取工具

6.6.2 innochecksum — 离线 InnoDB 文件校验工具

6.6.3 myisam_ftdump — 显示全文索引信息

6.6.4 myisamchk — MyISAM 表维护工具

6.6.5 myisamlog — 显示 MyISAM 日志文件内容

6.6.6 myisampack — 生成压缩的只读 MyISAM 表

6.6.7 mysql_config_editor — MySQL 配置工具

6.6.8 mysql_migrate_keyring — 密钥环键迁移实用程序

6.6.9 mysqlbinlog — 用于处理二进制日志文件的实用程序

6.6.10 mysqldumpslow — 汇总慢查询日志文件

6.7 程序开发工具

6.7.1 mysql_config — 显示用于编译客户端的选项

6.7.2 my_print_defaults — 显示选项文件中的选项

6.8 杂项程序

6.8.1 lz4_decompress — 解压缩 mysqlpump LZ4 压缩输出

6.8.2 perror — 显示 MySQL 错误消息信息

6.8.3 zlib_decompress — 解压缩 mysqlpump ZLIB 压缩输出

6.9 环境变量

6.10 MySQL 中的 Unix 信号处理

本章简要概述了由 Oracle Corporation 提供的 MySQL 命令行程序。它还讨论了在运行这些程序时指定选项的一般语法。大多数程序具有特定于其自身操作的选项，但所有这些程序的选项语法都是相似的。最后，本章提供了对各个程序的更详细描述，包括它们识别的选项。
