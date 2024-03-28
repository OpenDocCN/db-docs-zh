> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-key-types.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-key-types.html)

#### 8.4.4.13 支持的关键环关键类型和长度

MySQL 关键环支持不同类型（加密算法）和长度的关键：

+   可用的关键类型取决于安装了哪个关键环插件。

+   允许的关键长度受多个因素影响：

    +   一般关键环可加载函数接口限制（用于使用第 8.4.4.15 节，“通用关键环关键管理函数”中描述的关键环函数之一管理的关键），或来自后端实现的限制。这些长度限制可以根据关键操作类型而变化。

    +   除了一般限制外，单独的关键环插件可能对每种关键类型的关键长度施加限制。

表 8.32，“一般关键环关键长度限制”显示了一般关键长度限制。（`keyring_aws`的较低限制由 AWS KMS 接口而不是关键环函数强加。）对于关键环插件，表 8.33，“关键环插件关键类型和长度”显示了每个关键环插件允许的关键类型，以及任何插件特定的关键长度限制。对于大多数关键环组件，一般关键长度限制适用，没有关键类型限制。

注意

`component_keyring_oci`（类似于`keyring_oci`插件）只能生成大小为 16, 24 或 32 字节的`AES`类型的关键。

**表 8.32 一般关键环关键长度限制**

| 关键操作 | 最大关键长度 |
| --- | --- |
| 生成关键 | 16,384 字节（MySQL 8.0.18 之前为 2,048）；`keyring_aws`为 1,024 |
| 存储关键 | 16,384 字节（MySQL 8.0.18 之前为 2,048）；`keyring_aws`为 4,096 |
| 获取关键 | 16,384 字节（MySQL 8.0.18 之前为 2,048）；`keyring_aws`为 4,096 |

**表 8.33 关键环插件关键类型和长度**

| 插件名称 | 允许的关键类型 | 插件特定长度限制 |
| --- | --- | --- |
| `keyring_aws` | `AES``SECRET` | 16, 24 或 32 字节 None |
| `keyring_encrypted_file` | `AES``DSA``RSA``SECRET` | NoneNoneNoneNone |
| `keyring_file` | `AES``DSA``RSA``SECRET` | NoneNoneNoneNone |
| `keyring_hashicorp` | `AES``DSA``RSA``SECRET` | NoneNoneNoneNone |
| `keyring_oci` | `AES` | 16, 24 或 32 字节 |
| `keyring_okv` | `AES``SECRET` | 16, 24 或 32 字节 None |

`SECRET`关键类型，自 MySQL 8.0.19 起可用，旨在使用 MySQL 关键环对敏感数据进行通用存储，并受大多数关键环组件和关键环插件支持。关键环在存储和检索时将`SECRET`数据加密和解密为字节流。

涉及`SECRET`关键类型的示例关键环操作：

```sql
SELECT keyring_key_generate('MySecret1', 'SECRET', 20);
SELECT keyring_key_remove('MySecret1');

SELECT keyring_key_store('MySecret2', 'SECRET', 'MySecretData');
SELECT keyring_key_fetch('MySecret2');
SELECT keyring_key_length_fetch('MySecret2');
SELECT keyring_key_type_fetch('MySecret2');
SELECT keyring_key_remove('MySecret2');
```
