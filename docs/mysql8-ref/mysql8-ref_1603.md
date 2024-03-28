# 22.5.2 禁用 X 插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-disabling.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-disabling.html)

X 插件可以在启动时通过在 MySQL 配置文件中设置`mysqlx=0`，或者在启动 MySQL 服务器时传入`--mysqlx=0`或`--skip-mysqlx`来禁用。

或者，可以使用`-DWITH_MYSQLX=OFF` CMake 选项来编译不包含 X 插件的 MySQL 服务器。
