> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-service.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-service.html)

#### 7.6.9.2 密钥环服务

MySQL Server 支持一个密钥环服务，使内部组件和插件能够安全地存储敏感信息以供以后检索。MySQL 发行版提供了一个可在两个级别访问的密钥环接口：

+   在 SQL 级别，作为一组可加载函数，每个函数映射到对服务例程的调用。

+   作为 C 语言接口，可作为服务器插件或可加载函数的插件服务调用。

本节描述如何使用密钥环服务函数在 MySQL 密钥环中存储、检索和删除密钥。有关使用函数的 SQL 接口的信息，请参阅 Section 8.4.4.15, “通用密钥环密钥管理函数”。有关一般密钥环信息，请参阅 Section 8.4.4, “MySQL 密钥环”。

密钥环服务使用启用的底层密钥环插件，如果有的话。如果没有启用密钥环插件，密钥环服务调用将失败。

密钥库中的“记录”由数据（密钥本身）和通过该标识符访问密钥的唯一标识符组成。标识符有两部分：

+   `key_id`：密钥 ID 或名称。以 `mysql_` 开头的 `key_id` 值由 MySQL Server 保留。

+   `user_id`：会话有效用户 ID。如果没有用户上下文，此值可以为 `NULL`。该值实际上不必是“用户”；其含义取决于应用程序。

    实现密钥环函数接口的函数将 `CURRENT_USER()` 的值作为 `user_id` 值传递给密钥环服务函数。

密钥环服务功能具有以下共同特点：

+   每个函数成功返回 0，失败返回 1。

+   `key_id` 和 `user_id` 参数形成一个唯一组合，指示密钥环中要使用的密钥。

+   `key_type` 参数提供有关密钥的附加信息，例如其加密方法或预期用途。

+   密钥环服务函数将密钥 ID、用户名、类型和值视为二进制字符串，因此比较区分大小写。例如，`MyKey` 和 `mykey` 的 ID 指的是不同的密钥。

这些密钥环服务函数可用：

+   `my_key_fetch()`

    从密钥环中解密并检索密钥，以及其类型。该函数为用于存储返回的密钥和密钥类型的缓冲区分配内存。当不再需要时，调用者应将内存清零或混淆，然后释放它。

    语法：

    ```sql
    bool my_key_fetch(const char *key_id, const char **key_type,
                      const char* user_id, void **key, size_t *key_len)
    ```

    参数：

    +   `key_id`、`user_id`：作为一对的以空字符结尾的字符串，形成一个唯一标识符，指示要获取哪个密钥。

    +   `key_type`：缓冲区指针的地址。函数将一个指向提供有关密钥的附加信息的以空字符结尾的字符串的指针存储到其中（在添加密钥时存储）。

    +   `key`：缓冲区指针的地址。函数将一个指向包含获取的密钥数据的缓冲区的指针存储到其中。

    +   `key_len`：函数将`*key`缓冲区的字节大小存储到一个变量的地址中。

    返回值：

    返回 0 表示成功，返回 1 表示失败。

+   `my_key_generate()`

    生成给定类型和长度的新随机密钥，并将其存储在密钥环中。密钥的长度为`key_len`，与从`key_id`和`user_id`形成的标识符相关联。类型和长度值必须与底层密钥环插件支持的值一致。请参阅第 8.4.4.13 节，“支持的密钥环密钥类型和长度”。

    语法：

    ```sql
    bool my_key_generate(const char *key_id, const char *key_type,
                         const char *user_id, size_t key_len)
    ```

    参数：

    +   `key_id`，`user_id`：作为一对形成密钥的唯一标识符的以空字符结尾的字符串。

    +   `key_type`：一个以空字符结尾的字符串，提供有关密钥的附加信息。

    +   `key_len`：要生成的密钥的字节大小。

    返回值：

    返回 0 表示成功，返回 1 表示失败。

+   `my_key_remove()`

    从密钥环中移除一个密钥。

    语法：

    ```sql
    bool my_key_remove(const char *key_id, const char* user_id)
    ```

    参数：

    +   `key_id`，`user_id`：作为一对形成要移除的密钥的唯一标识符的以空字符结尾的字符串。

    返回值：

    返回 0 表示成功，返回 1 表示失败。

+   `my_key_store()`

    对密钥进行混淆并存储在密钥环中。

    语法：

    ```sql
    bool my_key_store(const char *key_id, const char *key_type,
                      const char* user_id, void *key, size_t key_len)
    ```

    参数：

    +   `key_id`，`user_id`：作为一对形成要存储的密钥的唯一标识符的以空字符结尾的字符串。

    +   `key_type`：一个以空字符结尾的字符串，提供有关密钥的附加信息。

    +   `key`：包含要存储的密钥数据的缓冲区。

    +   `key_len`：`key`缓冲区的字节大小。

    返回值：

    返回 0 表示成功，返回 1 表示失败。
