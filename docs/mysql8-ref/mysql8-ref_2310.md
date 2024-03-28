> 原文：[`dev.mysql.com/doc/refman/8.0/en/ignoring-user.html`](https://dev.mysql.com/doc/refman/8.0/en/ignoring-user.html)

#### B.3.2.13 忽略用户

如果你遇到以下错误，意味着当**mysqld**启动或重新加载授权表时，在`user`表中发现了一个密码无效的账户。

`Found wrong password for user '*`some_user`*'@'*`some_host`*'; ignoring user`

因此，该账户被权限系统简单地忽略了。要解决这个问题，给账户分配一个新的有效密码。
