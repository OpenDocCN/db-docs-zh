# 7.6.8 密钥环代理桥插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/daemon-keyring-proxy-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/daemon-keyring-proxy-plugin.html)

MySQL Keyring 最初使用服务器插件实现了密钥库功能，但从 MySQL 8.0.24 开始开始过渡到使用组件基础架构。 过渡包括修改密钥环插件的底层实现以使用组件基础架构。 这是通过名为`daemon_keyring_proxy_plugin`的插件实现的，它充当插件和组件服务 API 之间的桥梁，并使密钥环插件可以继续使用而不会改变用户可见的特性。

`daemon_keyring_proxy_plugin`是内置的，无需安装或启用。
