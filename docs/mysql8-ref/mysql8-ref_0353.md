> 原文：[`dev.mysql.com/doc/refman/8.0/en/password-security-admin.html`](https://dev.mysql.com/doc/refman/8.0/en/password-security-admin.html)

#### 8.1.2.2 密码安全的管理员准则

数据库管理员应遵循以下准则以保护密码安全。

MySQL 将用户账户的密码存储在`mysql.user`系统表中。绝对不应该将对该表的访问权限授予任何非管理员账户。

可以设置账户密码过期，以便用户必须重置密码。参见第 8.2.15 节，“密码管理”，以及第 8.2.16 节，“服务器处理过期密码”。

可以使用`validate_password`插件来强制执行可接受密码的策略。参见第 8.4.3 节，“密码验证组件”。

一个用户如果有权限修改插件目录（`plugin_dir`系统变量的值）或指定插件目录位置的`my.cnf`文件，就可以替换插件并修改插件提供的功能，包括认证插件。

应该保护可能写入密码的文件，比如日志文件。参见第 8.1.2.3 节，“密码和日志记录”。
