> 原文：[`dev.mysql.com/doc/refman/8.0/en/version-tokens-elements.html`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-elements.html)

#### 7.6.6.1 版本令牌元素

版本令牌基于一个实现以下元素的插件库：

+   一个名为`version_tokens`的服务器端插件保存与服务器关联的版本令牌列表，并订阅语句执行事件的通知。`version_tokens`插件使用审计插件 API 来监视来自客户端的传入语句，并将每个客户端的会话特定版本令牌列表与服务器版本令牌列表进行匹配。如果匹配成功，插件允许语句通过，服务器继续处理。否则，插件向客户端返回错误，语句失败。

+   一组可加载函数提供了一个 SQL 级 API，用于操作和检查插件维护的服务器版本令牌列表。调用任何版本令牌函数都需要`VERSION_TOKEN_ADMIN`权限（或已弃用的`SUPER`权限）。

+   当`version_tokens`插件加载时，它定义了`VERSION_TOKEN_ADMIN`动态权限。这个权限可以授予函数的用户。

+   一个系统变量使客户端能够指定注册所需服务器状态的版本令牌列表。如果服务器在客户端发送语句时处于不同状态，则客户端会收到错误。
