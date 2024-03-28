# 20.5.3 事务一致性保证

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-consistency-guarantees.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-consistency-guarantees.html)

20.5.3.1 理解事务一致性保证

20.5.3.2 配置事务一致性保证

分布式系统（如 Group Replication）的一个重要影响是作为一个群体提供的一致性保证。换句话说，是分布在群体成员之间的事务全局同步的一致性。本节描述了 Group Replication 如何处理依赖于群体中发生的事件的一致性保证，以及如何最佳配置您的群体的一致性保证。
