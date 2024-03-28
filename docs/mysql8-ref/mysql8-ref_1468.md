> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-views.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-views.html)

#### 19.5.1.40 复制和视图

视图始终会被复制到副本中。视图是根据其自身名称进行过滤的，而不是根据它们所引用的表进行过滤。这意味着即使视图包含通常会被`replication-ignore-table`规则过滤掉的表，视图也可以被复制到副本中。因此，应该注意确保视图不会复制通常出于安全原因而被过滤的表数据。

使用基于语句的日志记录支持从表复制到同名视图，但在使用基于行的日志记录时不支持。在启用基于行的日志记录时尝试这样做会导致错误。
