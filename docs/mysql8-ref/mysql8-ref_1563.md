# 20.9.2 组复制状态变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-status-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-status-variables.html)

MySQL 8.0 支持一个提供有关组复制信息的状态变量。该变量在此处描述：

+   `group_replication_primary_member`

    在单主模式下运行时显示主成员的 UUID。如果组在多主模式下运行，则显示空字符串。

    警告

    `group_replication_primary_member`状态变量已被弃用，并计划在将来的版本中删除。

    查看第 20.1.3.1.2 节，“查找主”。
