> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-session-ssl-status.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-session-ssl-status.html)

#### 30.4.3.34 session_ssl_status 视图

对于每个连接，该视图显示 SSL 版本、密码和重用 SSL 会话的计数。

`session_ssl_status` 视图包含以下列：

+   `thread_id`

    连接的线程 ID。

+   `ssl_version`

    连接使用的 SSL 版本。

+   `ssl_cipher`

    连接使用的 SSL 密码。

+   `ssl_sessions_reused`

    连接中重用的 SSL 会话数。
