# 22.5.4 使用 X 插件与缓存 SHA-2 认证插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-sha2-cache-plugin.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-sha2-cache-plugin.html)

X 插件支持使用`caching_sha2_password`认证插件创建的 MySQL 用户账户。有关此插件的更多信息，请参阅 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。您可以使用 X 插件对这些账户进行身份验证，使用非 SSL 连接进行`SHA256_MEMORY`认证和使用 SSL 连接进行`PLAIN`认证。

虽然`caching_sha2_password`认证插件拥有一个认证缓存，但这个缓存不与 X 插件共享，因此 X 插件使用自己的认证缓存进行`SHA256_MEMORY`认证。X 插件认证缓存存储用户账户密码的哈希值，并且无法通过 SQL 访问。如果用户账户被修改或移除，相关条目将从缓存中移除。X 插件认证缓存由默认启用的`mysqlx_cache_cleaner`插件维护，没有相关的系统变量或状态变量。

在您可以使用非 SSL X 协议连接对使用`caching_sha2_password`认证插件的账户进行身份验证之前，该账户必须至少通过 SSL 进行一次 X 协议连接的认证，以向 X 插件认证缓存提供密码。一旦这个初始的 SSL 认证成功，就可以使用非 SSL X 协议连接。

可以通过使用选项`--mysqlx_cache_cleaner=0`启动 MySQL 服务器来禁用`mysqlx_cache_cleaner`插件。如果这样做，X 插件认证缓存将被禁用，因此在使用`SHA256_MEMORY`认证进行身份验证时，必须始终使用 SSL 进行 X 协议连接。
