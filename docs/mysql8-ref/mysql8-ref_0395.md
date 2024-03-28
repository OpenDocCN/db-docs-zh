> 原文：[`dev.mysql.com/doc/refman/8.0/en/native-pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/native-pluggable-authentication.html)

#### 8.4.1.1 本地可插拔认证

MySQL 包含一个实现本地认证的`mysql_native_password`插件；即，基于在可插拔认证引入之前使用的密码哈希方法的认证。

注意

从 MySQL 8.0.34 开始，`mysql_native_password`认证插件已被弃用，并可能在将来的 MySQL 版本中被移除。

以下表显示了服务器端和客户端端的插件名称。

**表 8.16 本地密码认证的插件和库名称**

| 插件或文件 | 插件或文件名 |
| --- | --- |
| 服务器端插件 | `mysql_native_password` |
| 客户端插件 | `mysql_native_password` |
| 库文件 | 无（插件已内置） |

以下各节提供了特定于本地可插拔认证的安装和使用信息：

+   安装本地可插拔认证

+   使用本地可插拔认证

有关 MySQL 中可插拔认证的一般信息，请参阅第 8.2.17 节，“可插拔认证”。

##### 安装本地可插拔认证

`mysql_native_password`插件存在于服务器和客户端形式中：

+   服务器端插件内置于服务器中，无需显式加载，并且无法通过卸载来禁用它。

+   客户端插件内置于`libmysqlclient`客户端库中，并可供链接到`libmysqlclient`的任何程序使用。

##### 使用本地可插拔认证

MySQL 客户端程序默认使用`mysql_native_password`。`--default-auth`选项可用作程序可以期望使用的客户端插件的提示：

```sql
$> mysql --default-auth=mysql_native_password ...
```
