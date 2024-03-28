# 20.8 升级群组复制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-upgrade.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-upgrade.html)

20.8.1 在群组中合并不同成员版本

20.8.2 群组复制离线升级

20.8.3 群组复制在线升级

本节解释了如何升级群组复制设置。升级群组成员的基本过程与升级独立实例的过程相同，请参阅 Chapter 3, *Upgrading MySQL*了解实际的升级过程和可用的类型。在原地升级和逻辑升级之间的选择取决于群组中存储的数据量。通常，原地升级更快，因此建议使用。您还应该参考 Section 19.5.3, “Upgrading a Replication Topology”。

在升级在线群组的过程中，为了最大化可用性，您可能需要在同一时间运行具有不同 MySQL 服务器版本的成员。群组复制包括兼容性策略，使您能够在升级过程中安全地将运行不同 MySQL 版本的成员合并到同一群组中。根据您的群组，这些策略的影响可能会影响您应该升级群组成员的顺序。有关详细信息，请参见 Section 20.8.1, “Combining Different Member Versions in a Group”。

如果您的群组可以完全脱机，请参见 Section 20.8.2, “群组复制离线升级”。如果您的群组需要保持在线，这在生产部署中很常见，请参见 Section 20.8.3, “群组复制在线升级”，了解升级群组的不同方法以最小化停机时间。
