# 14.2 可加载函数参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/loadable-function-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/loadable-function-reference.html)

下表列出了每个可在运行时加载的函数，并提供了每个函数的简短描述。有关列出内置函数和运算符的表格，请参见第 14.1 节，“内置函数和运算符参考”

有关可加载函数的一般信息，请参见第 7.7 节，“MySQL 服务器可加载函数”。

**表 14.2 可加载函数**

| 名称 | 描述 | 引入版本 | 废弃版本 |
| --- | --- | --- | --- |
| `asymmetric_decrypt()` | 使用私钥或公钥解密密文 |  |  |
| `asymmetric_derive()` | 从非对称密钥派生对称密钥 |  |  |
| `asymmetric_encrypt()` | 使用私钥或公钥加密明文 |  |  |
| `asymmetric_sign()` | 从摘要生成签名 |  |  |
| `asymmetric_verify()` | 验证签名是否匹配摘要 |  |  |
| `asynchronous_connection_failover_add_managed()` | 将一个受管组中的复制源服务器添加到源列表 | 8.0.23 |  |
| `asynchronous_connection_failover_add_source()` | 将一个复制源服务器添加到源列表 | 8.0.22 |  |
| `asynchronous_connection_failover_delete_managed()` | 从源列表中删除受管复制源服务器组 | 8.0.23 |  |
| `asynchronous_connection_failover_delete_source()` | 从源列表中删除一个复制源服务器 | 8.0.22 |  |
| `audit_api_message_emit_udf()` | 向审计日志添加消息事件 |  |  |
| `audit_log_encryption_password_get()` | 获取审计日志加密密码 |  |  |
| `audit_log_encryption_password_set()` | 设置审计日志加密密码 |  |  |
| `audit_log_filter_flush()` | 刷新审计日志过滤表 |  |  |
| `audit_log_filter_remove_filter()` | 移除审计日志过滤器 |  |  |
| `audit_log_filter_remove_user()` | 从用户中取消分配审计日志过滤器 |  |  |
| `audit_log_filter_set_filter()` | 定义审计日志过滤器 |  |  |
| `audit_log_filter_set_user()` | 为用户分配审计日志过滤器 |  |  |
| `audit_log_read()` | 返回审计日志记录 |  |  |
| `audit_log_read_bookmark()` | 最近审计日志事件的书签 |  |  |
| `audit_log_rotate()` | 旋转审计日志文件 |  |  |
| `create_asymmetric_priv_key()` | 创建私钥 |  |  |
| `create_asymmetric_pub_key()` | 创建公钥 |  |  |
| `create_dh_parameters()` | 生成共享 DH 密钥 |  |  |
| `create_digest()` | 从字符串生成摘要 |  |  |
| `firewall_group_delist()` | 从防火墙组配置文件中移除帐户 | 8.0.23 |  |
| `firewall_group_enlist()` | 将帐户添加到防火墙组配置文件 | 8.0.23 |  |
| `flush_rewrite_rules()` | 将 rewrite_rules 表加载到 Rewriter 缓存中 |  |  |
| `gen_blacklist()` | 执行字典术语替换 |  | 8.0.23 |
| `gen_blocklist()` | 执行字典术语替换 | 8.0.33 |  |
| `gen_blocklist()` | 执行字典术语替换 | 8.0.23 |  |
| `gen_dictionary()` | 从字典中返回随机术语 | 8.0.33 |  |
| `gen_dictionary_drop()` | 从注册表中移除字典 |  |  |
| `gen_dictionary_load()` | 将字典加载到注册表中 |  |  |
| `gen_dictionary()` | 从字典中返回随机术语 |  |  |
| `gen_range()` | 在范围内生成随机数 | 8.0.33 |  |
| `gen_range()` | 在范围内生成随机数 |  |  |
| `gen_rnd_canada_sin()` | 生成随机加拿大社会保险号码 | 8.0.33 |  |
| `gen_rnd_email()` | 生成随机电子邮件地址 | 8.0.33 |  |
| `gen_rnd_email()` | 生成随机电子邮件地址 |  |  |
| `gen_rnd_iban()` | 生成随机国际银行帐号 | 8.0.33 |  |
| `gen_rnd_pan()` | 生成随机支付卡主帐号 | 8.0.33 |  |
| `gen_rnd_pan()` | 生成随机支付卡主帐号 |  |  |
| `gen_rnd_ssn()` | 生成随机美国社会安全号码 | 8.0.33 |  |
| `gen_rnd_ssn()` | 生成随机美国社会安全号码 |  |  |
| `gen_rnd_uk_nin()` | 生成随机英国国民保险号码 | 8.0.33 |  |
| `gen_rnd_us_phone()` | 生成随机美国电话号码 | 8.0.33 |  |
| `gen_rnd_us_phone()` | 生成随机美国电话号码 |  |  |
| `gen_rnd_uuid()` | 生成随机通用唯一标识符 | 8.0.33 |  |
| `group_replication_disable_member_action()` | 启用成员操作，使成员在指定情况下不执行 |  |  |
| `group_replication_enable_member_action()` | 在指定情况下启用成员操作 |  |  |
| `group_replication_get_communication_protocol()` | 返回 Group Replication 协议版本 |  |  |
| `group_replication_get_write_concurrency()` | 返回可并行执行的最大共识实例数 |  |  |
| `group_replication_reset_member_actions()` | 将成员操作配置重置为默认设置 |  |  |
| `group_replication_set_as_primary()` | 将组成员指定为新主 |  |  |
| `group_replication_set_communication_protocol()` | 设置组复制协议版本 |  |  |
| `group_replication_set_write_concurrency()` | 设置可以并行执行的最大一致性实例数 |  |  |
| `group_replication_switch_to_multi_primary_mode()` | 将组从单主模式切换到多主模式 |  |  |
| `group_replication_switch_to_single_primary_mode()` | 将组从多主模式切换到单主模式 |  |  |
| `keyring_aws_rotate_cmk()` | 旋转 AWS 客户主密钥 |  |  |
| `keyring_aws_rotate_keys()` | 旋转 keyring_aws 存储文件中的钥匙 |  |  |
| `keyring_hashicorp_update_config()` | 导致运行时 keyring_hashicorp 重新配置 |  |  |
| `keyring_key_fetch()` | 获取钥匙环钥匙值 |  |  |
| `keyring_key_generate()` | 生成随机钥匙环钥匙 |  |  |
| `keyring_key_length_fetch()` | 返回钥匙环钥匙长度 |  |  |
| `keyring_key_remove()` | 移除钥匙环钥匙 |  |  |
| `keyring_key_store()` | 在钥匙环中存储钥匙 |  |  |
| `keyring_key_type_fetch()` | 返回钥匙环钥匙类型 |  |  |
| `load_rewrite_rules()` | 重写插件辅助例程 |  |  |
| `mask_canada_sin()` | 遮蔽加拿大社会保险号码 | 8.0.33 |  |
| `mask_iban()` | 遮蔽国际银行账号 | 8.0.33 |  |
| `mask_inner()` | 遮蔽字符串的内部部分 | 8.0.33 |  |
| `mask_inner()` | 遮蔽字符串的内部部分 |  |  |
| `mask_outer()` | 遮蔽字符串的左右部分 | 8.0.33 |  |
| `mask_outer()` | 遮蔽字符串的左右部分 |  |  |
| `mask_pan()` | 遮蔽支付卡主帐号部分字符串 | 8.0.33 |  |
| `mask_pan()` | 遮蔽支付卡主帐号部分字符串 |  |  |
| `mask_pan_relaxed()` | 遮蔽支付卡主帐号部分字符串 | 8.0.33 |  |
| `mask_pan_relaxed()` | 遮蔽支付卡主帐号部分字符串 |  |  |
| `mask_ssn()` | 遮蔽美国社会安全号码 | 8.0.33 |  |
| `mask_ssn()` | 遮蔽美国社会安全号码 |  |  |
| `mask_uk_nin()` | 遮蔽英国国民保险号码 | 8.0.33 |  |
| `mask_uuid()` | 遮蔽字符串的通用唯一标识符部分 | 8.0.33 |  |
| `masking_dictionary_remove()` | 从数据库表中删除字典 | 8.0.33 |  |
| `masking_dictionary_term_add()` | 向字典中添加新术语 | 8.0.33 |  |
| `masking_dictionary_term_remove()` | 从字典中删除现有术语 | 8.0.33 |  |
| `mysql_firewall_flush_status()` | 重置防火墙状态变量 |  |  |
| `mysql_query_attribute_string()` | 获取查询属性值 | 8.0.23 |  |
| `normalize_statement()` | 将 SQL 语句规范化为摘要形式 |  |  |
| `read_firewall_group_allowlist()` | 更新防火墙组配置文件记录语句缓存 | 8.0.23 |  |
| `read_firewall_groups()` | 更新防火墙组配置文件缓存 | 8.0.23 |  |
| `read_firewall_users()` | 更新防火墙账户配置文件缓存 |  | 8.0.26 |
| `read_firewall_whitelist()` | 更新防火墙账户配置文件记录语句缓存 |  | 8.0.26 |
| `service_get_read_locks()` | 获取锁定服务共享锁 |  |  |
| `service_get_write_locks()` | 获取锁定服务独占锁 |  |  |
| `service_release_locks()` | 释放锁定服务锁 |  |  |
| `set_firewall_group_mode()` | 建立防火墙组配置操作模式 | 8.0.23 |  |
| `set_firewall_mode()` | 建立防火墙账户配置操作模式 |  | 8.0.26 |
| `version_tokens_delete()` | 从版本令牌列表中删除令牌 |  |  |
| `version_tokens_edit()` | 修改版本令牌列表 |  |  |
| `version_tokens_lock_exclusive()` | 获取版本令牌的独占锁 |  |  |
| `version_tokens_lock_shared()` | 获取版本令牌的共享锁 |  |  |
| `version_tokens_set()` | 设置版本令牌列表 |  |  |
| `version_tokens_show()` | 返回版本令牌列表 |  |  |
| `version_tokens_unlock()` | 释放版本令牌锁 |  |  |
| 名称 | 描述 | 引入版本 | 废弃版本 |
