> 原文：[`dev.mysql.com/doc/refman/8.0/en/nonpersistible-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/nonpersistible-system-variables.html)

#### 7.1.9.4 不可持久化和受持久化限制的系统变量

`SET PERSIST` 和 `SET PERSIST_ONLY` 允许全局系统变量被持久化到数据目录中的 `mysqld-auto.cnf` 选项文件中（参见 Section 15.7.6.1, “变量赋值的 SET 语法”）。然而，并非所有系统变量都可以被持久化，或者只能在某些限制条件下被持久化。以下是系统变量可能为不可持久化或受持久化限制的一些原因：

+   会话系统变量无法被持久化。会话变量无法在服务器启动时设置，因此没有理由将它们持久化。

+   全局系统变量可能涉及敏感数据，因此只能由直接访问服务器主机的用户设置。

+   全局系统变量可能是只读的（即只能由服务器设置）。在这种情况下，无论是在服务器启动时还是在运行时，用户都无法设置它。

+   全局系统变量可能仅供内部使用。

不可持久化的系统变量无论如何都不能被持久化。从 MySQL 8.0.14 开始，受持久化限制的系统变量可以通过 `SET PERSIST_ONLY` 被持久化，但只能由满足以下条件的用户进行：

+   `persist_only_admin_x509_subject` 系统变量设置为 SSL 证书 X.509 主题值。

+   用户使用加密连接连接到服务器，并提供具有指定主题值的 SSL 证书。

+   用户具有足够的权限使用 `SET PERSIST_ONLY`（参见 Section 7.1.9.1, “系统变量权限”）。

例如，`protocol_version` 是只读的，只能由服务器设置，因此无论如何都不能持久化。另一方面，`bind_address` 是受持久化限制的，因此只有符合前述条件的用户才能设置它。

以下系统变量是不可持久化的。此列表可能会随着持续开发而更改。

```sql
audit_log_current_session
audit_log_filter_id
caching_sha2_password_digest_rounds
character_set_system
core_file
have_statement_timeout
have_symlink
hostname
innodb_version
keyring_hashicorp_auth_path
keyring_hashicorp_ca_path
keyring_hashicorp_caching
keyring_hashicorp_commit_auth_path
keyring_hashicorp_commit_ca_path
keyring_hashicorp_commit_caching
keyring_hashicorp_commit_role_id
keyring_hashicorp_commit_server_url
keyring_hashicorp_commit_store_path
keyring_hashicorp_role_id
keyring_hashicorp_secret_id
keyring_hashicorp_server_url
keyring_hashicorp_store_path
large_files_support
large_page_size
license
locked_in_memory
log_bin
log_bin_basename
log_bin_index
lower_case_file_system
ndb_version
ndb_version_string
persist_only_admin_x509_subject
persisted_globals_load
protocol_version
relay_log_basename
relay_log_index
server_uuid
skip_external_locking
system_time_zone
version_comment
version_compile_machine
version_compile_os
version_compile_zlib
```

受限制的持久化系统变量是那些只读的变量，可以在命令行或选项文件中设置，除了`persist_only_admin_x509_subject`和`persisted_globals_load`之外。此列表可能会随着持续开发而更改。

```sql
audit_log_file
audit_log_format
auto_generate_certs
basedir
bind_address
caching_sha2_password_auto_generate_rsa_keys
caching_sha2_password_private_key_path
caching_sha2_password_public_key_path
character_sets_dir
daemon_memcached_engine_lib_name
daemon_memcached_engine_lib_path
daemon_memcached_option
datadir
default_authentication_plugin
ft_stopword_file
init_file
innodb_buffer_pool_load_at_startup
innodb_data_file_path
innodb_data_home_dir
innodb_dedicated_server
innodb_directories
innodb_force_load_corrupted
innodb_log_group_home_dir
innodb_page_size
innodb_read_only
innodb_temp_data_file_path
innodb_temp_tablespaces_dir
innodb_undo_directory
innodb_undo_tablespaces
keyring_encrypted_file_data
keyring_encrypted_file_password
lc_messages_dir
log_error
mecab_rc_file
named_pipe
pid_file
plugin_dir
port
relay_log
relay_log_info_file
replica_load_tmpdir
secure_file_priv
sha256_password_auto_generate_rsa_keys
sha256_password_private_key_path
sha256_password_public_key_path
shared_memory
shared_memory_base_name
skip_networking
slave_load_tmpdir
socket
ssl_ca
ssl_capath
ssl_cert
ssl_crl
ssl_crlpath
ssl_key
tmpdir
version_tokens_session_number
```

要配置服务器以启用持久化受限制的系统变量，请使用以下步骤：

1.  确保 MySQL 配置支持加密连接。参见第 8.3.1 节，“配置 MySQL 使用加密连接”。

1.  指定一个 SSL 证书 X.509 主题值，表示能够持久化受限制的系统变量，并生成具有该主题的证书。参见第 8.3.3 节，“创建 SSL 和 RSA 证书和密钥”。

1.  启动服务器时，将`persist_only_admin_x509_subject`设置为指定的主题值。例如，在服务器的`my.cnf`文件中添加以下行：

    ```sql
    [mysqld]
    persist_only_admin_x509_subject="*subject-value*"
    ```

    主题值的格式与用于`CREATE USER ... REQUIRE SUBJECT`相同。参见第 15.7.1.3 节，“CREATE USER Statement”。

    您必须直接在 MySQL 服务器主机上执行此步骤，因为`persist_only_admin_x509_subject`本身无法在运行时持久化。

1.  重新启动服务器。

1.  将具有指定主题值的 SSL 证书分发给被允许持久化受限制系统变量的用户。

假设`myclient-cert.pem`是要供客户端使用的 SSL 证书，该证书可以持久化受限制的系统变量。使用**openssl**命令显示证书内容：

```sql
$> openssl x509 -text -in myclient-cert.pem
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 2 (0x2)
    Signature Algorithm: md5WithRSAEncryption
        Issuer: C=US, ST=IL, L=Chicago, O=MyOrg, OU=CA, CN=MyCN
        Validity
            Not Before: Oct 18 17:03:03 2018 GMT
            Not After : Oct 15 17:03:03 2028 GMT
        Subject: C=US, ST=IL, L=Chicago, O=MyOrg, OU=client, CN=MyCN
...
```

**openssl**输出显示证书主题值为：

```sql
C=US, ST=IL, L=Chicago, O=MyOrg, OU=client, CN=MyCN
```

要为 MySQL 指定主题，使用以下格式：

```sql
/C=US/ST=IL/L=Chicago/O=MyOrg/OU=client/CN=MyCN
```

在服务器的`my.cnf`文件中配置主题值：

```sql
[mysqld]
persist_only_admin_x509_subject="/C=US/ST=IL/L=Chicago/O=MyOrg/OU=client/CN=MyCN"
```

重新启动服务器以使新配置生效。

将 SSL 证书（以及任何其他相关 SSL 文件）分发给适当的用户。然后，此用户使用证书和建立加密连接所需的任何其他 SSL 选项连接到服务器。

要使用 X.509，客户端必须指定`--ssl-key`和`--ssl-cert`选项进行连接。建议但不是必须还指定`--ssl-ca`，以便验证服务器提供的公共证书。例如：

```sql
$> mysql --ssl-key=myclient-key.pem --ssl-cert=myclient-cert.pem --ssl-ca=mycacert.pem
```

假设用户具有足够的特权来使用`SET PERSIST_ONLY`，则可以像这样持久化受限制的系统变量：

```sql
mysql> SET PERSIST_ONLY socket = '/tmp/mysql.sock';
Query OK, 0 rows affected (0.00 sec)
```

如果服务器未配置为启用持久化受限制系统变量，或用户不满足该功能所需的条件，则会发生错误：

```sql
mysql> SET PERSIST_ONLY socket = '/tmp/mysql.sock';
ERROR 1238 (HY000): Variable 'socket' is a non persistent read only variable
```
