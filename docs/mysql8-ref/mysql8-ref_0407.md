> 原文：[`dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html)

#### 8.4.1.13 可插拔认证系统变量

除非安装了适当的服务器端插件，否则这些变量不可用：

+   `authentication_ldap_sasl` 用于形式为 `authentication_ldap_sasl_*`xxx`*` 的系统变量

+   `authentication_ldap_simple` 用于形式为 `authentication_ldap_simple_*`xxx`*` 的系统变量

**表 8.29 认证插件系统变量摘要**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| [authentication_fido_rp_id](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_fido_rp_id) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_kerberos_service_key_tab](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_kerberos_service_key_tab) | 是 | 是 | 是 |  | 全局 | 否 |
| [authentication_kerberos_service_principal](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_kerberos_service_principal) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_auth_method_name](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_auth_method_name) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_bind_base_dn](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_bind_base_dn) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_bind_root_dn](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_bind_root_dn) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_bind_root_pwd](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_bind_root_pwd) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_ca_path](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_ca_path) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_group_search_attr](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_group_search_attr) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_group_search_filter](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_group_search_filter) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_init_pool_size](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_init_pool_size) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_log_status](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_log_status) | 是 | 是 | 是 |  | 全局 | 是 |
| [authentication_ldap_sasl_max_pool_size](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication-system-variables.html#sysvar_authentication_ldap_sasl_max_pool_size) | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_referral | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_server_host | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_server_port | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_tls | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_sasl_user_search_attr | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_auth_method_name | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_bind_base_dn | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_bind_root_dn | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_bind_root_pwd | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_ca_path | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_group_search_attr | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_group_search_filter | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_init_pool_size | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_log_status | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_max_pool_size | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_referral | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_server_host | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_server_port | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_tls | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_ldap_simple_user_search_attr | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_policy | 是 | 是 | 是 |  | 全局 | 是 |
| authentication_windows_log_level | 是 | 是 | 是 |  | 全局 | 否 |
| authentication_windows_use_principal_name | 是 | 是 | 是 |  | 全局 | 否 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |

+   `authentication_fido_rp_id`

    | 命令行格式 | `--authentication-fido-rp-id=value` |
    | --- | --- |
    | 引入版本 | 8.0.27 |
    | 弃用 | 8.0.35 |
    | 系统变量 | `authentication_fido_rp_id` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 默认值 | `MySQL` |

    此变量指定用于 FIDO 设备注册和 FIDO 认证的依赖方 ID。如果尝试进行 FIDO 认证且此值与 FIDO 设备预期的值不符，则设备会认为自己未与正确的服务器通信，从而导致错误。最大值长度为 255 个字符。

    注意

    截至 MySQL 8.0.35 版本，此插件变量已被弃用，并可能在未来的 MySQL 版本中移除。

+   `authentication_kerberos_service_key_tab`

    | 命令行格式 | `--authentication-kerberos-service-key-tab=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `authentication_kerberos_service_key_tab` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 文件名 |
    | 默认值 | `datadir/mysql.keytab` |

    包含用于认证从客户端接收的 MySQL 服务票据的 Kerberos 服务密钥的服务器端密钥表（“keytab”）文件的名称。文件名应给出为绝对路径名。如果未设置此变量，则默认值为数据目录中的 `mysql.keytab`。

    文件必须存在并包含服务主体名称（SPN）的有效密钥，否则客户端的身份验证将失败。 （SPN 和相同密钥也必须在 Kerberos 服务器中创建。） 该文件可能包含多个服务主体名称及其相应的密钥组合。

    文件必须由 Kerberos 服务器管理员生成，并复制到 MySQL 服务器可访问的位置。 可以使用以下命令验证文件是否正确并已正确复制：

    ```sql
    klist -k *file_name*
    ```

    有关 keytab 文件的信息，请参阅 [`web.mit.edu/kerberos/krb5-latest/doc/basic/keytab_def.html`](https://web.mit.edu/kerberos/krb5-latest/doc/basic/keytab_def.html)。

+   `authentication_kerberos_service_principal`

    | 命令行格式 | `--authentication-kerberos-service-principal=name` |
    | --- | --- |
    | 引入 | 8.0.26 |
    | 系统变量 | `authentication_kerberos_service_principal` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `mysql/host_name@realm_name` |

    MySQL 服务器发送给客户端的 Kerberos 服务主体名称（SPN）。

    该值由服务名称（`mysql`）、主机名和领域名组成。 默认值为 `mysql/*`host_name`*@*`realm_name`*`。 服务主体名称中的领域使得可以检索到确切的服务密钥。

    要使用非默认值，请使用相同格式设置该值。 例如，要使用主机名为 `krbauth.example.com` 和领域为 `MYSQL.LOCAL`，请将 `authentication_kerberos_service_principal` 设置为 `mysql/krbauth.example.com@MYSQL.LOCAL`。

    服务主体名称和服务密钥必须已经存在于由 KDC 服务器管理的数据库中。

    可能存在仅由领域名称不同的服务主体名称。

+   `authentication_ldap_sasl_auth_method_name`

    | 命令行格式 | `--authentication-ldap-sasl-auth-method-name=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_auth_method_name` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `SCRAM-SHA-1` |
    | 有效值（≥ 8.0.23） | `SCRAM-SHA-1``SCRAM-SHA-256``GSSAPI` |
    | 有效值（≥ 8.0.20, ≤ 8.0.22） | `SCRAM-SHA-1``GSSAPI` |
    | 有效值（≤ 8.0.19） | `SCRAM-SHA-1` |

    对于 SASL LDAP 身份验证，这是身份验证方法的名称。身份验证插件与 LDAP 服务器之间的通信根据这种身份验证方法进行，以确保密码安全。

    允许这些身份验证方法值：

    +   `SCRAM-SHA-1`：使用 SASL 挑战-响应机制。

        客户端端的 `authentication_ldap_sasl_client` 插件与 SASL 服务器通信，使用密码创建挑战并获取 SASL 请求缓冲区，然后将此缓冲区传递给服务器端的 `authentication_ldap_sasl` 插件。客户端端和服务器端的 SASL LDAP 插件使用 SASL 消息来安全传输凭据，以避免在 MySQL 客户端和服务器之间发送明文密码。

    +   `SCRAM-SHA-256`：使用 SASL 挑战-响应机制。

        这种方法类似于 `SCRAM-SHA-1`，但更安全。它在 MySQL 8.0.23 及更高版本中可用。它需要使用 Cyrus SASL 2.1.27 或更高版本构建的 OpenLDAP 服务器。

    +   `GSSAPI`：使用 Kerberos，一种无密码且基于票据的协议。

        GSSAPI/Kerberos 是 MySQL 客户端和服务器仅在 Linux 上支持的身份验证方法。在 Linux 环境中，应用程序使用默认启用 Kerberos 的 Microsoft Active Directory 访问 LDAP 时，这是非常有用的。

        客户端端的 `authentication_ldap_sasl_client` 插件使用来自 Kerberos 的票据授予票据（TGT）获取服务票据，但不直接使用 LDAP 服务。服务器端的 `authentication_ldap_sasl` 插件在客户端插件和 LDAP 服务器之间路由 Kerberos 消息。然后，服务器端插件使用获取的凭据与 LDAP 服务器通信，解释 LDAP 身份验证消息并检索 LDAP 组。

+   `authentication_ldap_sasl_bind_base_dn`

    | 命令行格式 | `--authentication-ldap-sasl-bind-base-dn=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_bind_base_dn` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于 SASL LDAP 身份验证，基本区分名称（DN）。此变量可用于通过在搜索树中的特定位置（“基本”）锚定它们来限制搜索范围。

    假设一组 LDAP 用户条目的成员具有以下形式：

    ```sql
    uid=*user_name*,ou=People,dc=example,dc=com
    ```

    另一组 LDAP 用户条目的成员具有以下形式：

    ```sql
    uid=*user_name*,ou=Admin,dc=example,dc=com
    ```

    然后根据不同的基本 DN 值进行搜索：

    +   如果基本 DN 是 `ou=People,dc=example,dc=com`：搜索仅在第一组中找到用户条目。

    +   如果基本 DN 是 `ou=Admin,dc=example,dc=com`：搜索仅在第二组中找到用户条目。

    +   如果基本 DN 是 `ou=dc=example,dc=com`：搜索会在第一组或第二组中找到用户条目。

    一般来说，更具体的基本 DN 值会导致更快的搜索，因为它们限制了搜索范围。

+   `authentication_ldap_sasl_bind_root_dn`

    | 命令行格式 | `--authentication-ldap-sasl-bind-root-dn=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_bind_root_dn` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于 SASL LDAP 认证，根区分名称（DN）。此变量与 `authentication_ldap_sasl_bind_root_pwd` 一起用作用于执行搜索的目的的 LDAP 服务器进行身份验证的凭据。身份验证使用一个或两个 LDAP 绑定操作，具体取决于 MySQL 帐户是否指定了 LDAP 用户 DN：

    +   如果帐户没有指定用户 DN：`authentication_ldap_sasl` 使用 `authentication_ldap_sasl_bind_root_dn` 和 `authentication_ldap_sasl_bind_root_pwd` 执行初始 LDAP 绑定。（这两个默认为空，所以如果它们没有设置，LDAP 服务器必须允许匿名连接。）生成的绑定 LDAP 句柄用于根据客户端用户名搜索用户 DN。`authentication_ldap_sasl` 使用用户 DN 和客户端提供的密码执行第二次绑定。

    +   如果帐户没有指定用户 DN：在这种情况下，第一个绑定操作是不必要的。`authentication_ldap_sasl` 使用用户 DN 和客户端提供的密码执行单个绑定。这比如果 MySQL 帐户没有指定 LDAP 用户 DN 要快。

+   `authentication_ldap_sasl_bind_root_pwd`

    | 命令行格式 | `--authentication-ldap-sasl-bind-root-pwd=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_bind_root_pwd` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于 SASL LDAP 认证，根分隔名的密码。此变量与`authentication_ldap_sasl_bind_root_dn`一起使用。请参阅该变量的描述。

+   `authentication_ldap_sasl_ca_path`

    | 命令行格式 | `--authentication-ldap-sasl-ca-path=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_ca_path` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于 SASL LDAP 认证，证书颁发机构文件的绝对路径。如果希望认证插件执行 LDAP 服务器证书的验证，则指定此文件。

    注意

    除了将`authentication_ldap_sasl_ca_path`变量设置为文件名外，还必须将适当的证书颁发机构证书添加到文件中，并启用`authentication_ldap_sasl_tls`系统变量。这些变量可以设置以覆盖默认的 OpenLDAP TLS 配置；请参阅 LDAP Pluggable Authentication and ldap.conf

+   `authentication_ldap_sasl_group_search_attr`

    | 命令行格式 | `--authentication-ldap-sasl-group-search-attr=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_group_search_attr` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `cn` |

    对于 SASL LDAP 认证，指定 LDAP 目录条目中指定组名的属性名称。如果`authentication_ldap_sasl_group_search_attr`具有默认值`cn`，则搜索将返回`cn`值作为组名。例如，如果具有`uid`值为`user1`的 LDAP 条目具有`cn`属性为`mygroup`，则搜索`user1`将返回`mygroup`作为组名。

    如果您不想进行组或代理认证，则此变量应为空字符串。

    如果组搜索属性是`isMemberOf`，LDAP 认证直接检索用户属性`isMemberOf`的值，并将其分配为组信息。如果组搜索属性不是`isMemberOf`，LDAP 认证将搜索用户是成员的所有组。（后者是默认行为。）此行为基于 LDAP 组信息可以以两种方式存储：1）组条目可以具有名为`memberUid`或`member`的属性，其值为用户名；2）用户条目可以具有名为`isMemberOf`的属性，其值为组名。

+   `authentication_ldap_sasl_group_search_filter`

    | 命令行格式 | `--authentication-ldap-sasl-group-search-filter=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_group_search_filter` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `(&#124;(&(objectClass=posixGroup)(memberUid=%s))(&(objectClass=group)(member=%s)))` |

    对于 SASL LDAP 认证，自定义组搜索过滤器。

    搜索过滤器值可以包含`{UA}`和`{UD}`符号，分别表示用户名和完整用户 DN。例如，`{UA}`会被替换为用户名，比如`"admin"`，而`{UD}`会被替换为完整 DN，比如`"uid=admin,ou=People,dc=example,dc=com"`。以下是默认值，支持 OpenLDAP 和 Active Directory：

    ```sql
    (|(&(objectClass=posixGroup)(memberUid={UA}))
      (&(objectClass=group)(member={UD})))
    ```

    在某些情况下，对于用户场景，`memberOf`是一个简单的用户属性，不包含任何组信息。为了增加灵活性，可以在组搜索属性前使用可选的`{GA}`前缀。任何带有`{GA}`前缀的组属性都被视为具有组名的用户属性。例如，对于值`{GA}MemberOf`，如果组值是 DN，则从组 DN 中返回第一个属性值作为组名。

+   `authentication_ldap_sasl_init_pool_size`

    | 命令行格式 | `--authentication-ldap-sasl-init-pool-size=#` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_init_pool_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `32767` |
    | 单位 | 连接 |

    对于 SASL LDAP 认证，连接到 LDAP 服务器的连接池的初始大小。根据对 LDAP 服务器的平均并发认证请求数量选择此变量的值。

    插件使用`authentication_ldap_sasl_init_pool_size`和`authentication_ldap_sasl_max_pool_size`一起进行连接池管理：

    +   当认证插件初始化时，它会创建`authentication_ldap_sasl_init_pool_size`个连接，除非`authentication_ldap_sasl_max_pool_size=0`以禁用连接池。

    +   如果插件在当前连接池中没有空闲连接时收到认证请求，插件可以创建一个新连接，最多达到`authentication_ldap_sasl_max_pool_size`所指定的最大连接池大小。

    +   如果插件在连接池已经达到最大值且没有空闲连接时收到请求，认证将失败。

    +   当插件卸载时，它会关闭所有连接池中的连接。

    对插件系统变量设置的更改可能不会对已经在池中的连接产生影响。例如，修改 LDAP 服务器主机、端口或 TLS 设置不会影响现有连接。但是，如果原始变量值无效且连接池无法初始化，则插件会尝试为下一个 LDAP 请求重新初始化池。在这种情况下，新的系统变量值将用于重新初始化尝试。

    如果`authentication_ldap_sasl_max_pool_size=0`以禁用连接池，插件打开的每个 LDAP 连接都使用那时系统变量的值。

+   `authentication_ldap_sasl_log_status`

    | 命令行格式 | `--authentication-ldap-sasl-log-status=#` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_log_status` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值（≥ 8.0.18） | `6` |
    | 最大值（≤ 8.0.17） | `5` |

    对于 SASL LDAP 认证，写入错误日志的消息的日志级别。以下表显示了允许的级别值及其含义。

    **表 8.30 authentication_ldap_sasl_log_status 的日志级别**

    | 选项值 | 记录的消息类型 |
    | --- | --- |
    | `1` | 没有消息 |
    | `2` | 错误消息 |
    | `3` | 错误和警告消息 |
    | `4` | 错误、警告和信息消息 |
    | `5` | 与前一级别相同，加上来自 MySQL 的调试消息 |
    | `6` | 与前一级别相同，加上来自 LDAP 库的调试消息 |

    从 MySQL 8.0.18 开始提供日志级别 6。

    在客户端端，可以通过设置`AUTHENTICATION_LDAP_CLIENT_LOG`环境变量将消息记录到标准输出。允许的默认值与`authentication_ldap_sasl_log_status`相同。

    `AUTHENTICATION_LDAP_CLIENT_LOG`环境变量仅适用于 SASL LDAP 认证。对于简单 LDAP 认证，它不起作用，因为在这种情况下，客户端插件是`mysql_clear_password`，它对 LDAP 操作一无所知。

+   `authentication_ldap_sasl_max_pool_size`

    | 命令行格式 | `--authentication-ldap-sasl-max-pool-size=#` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_max_pool_size` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `0` |
    | 最大值 | `32767` |
    | 单位 | 连接 |

    对于 SASL LDAP 认证，连接到 LDAP 服务器的连接池的最大大小。要禁用连接池，请将此变量设置为 0。

    此变量与`authentication_ldap_sasl_init_pool_size`一起使用。请参阅该变量的描述。

+   `authentication_ldap_sasl_referral`

    | 命令行格式 | `--authentication-ldap-sasl-referral[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `authentication_ldap_sasl_referral` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    对于 SASL LDAP 认证，是否启用 LDAP 搜索引荐。请参阅 LDAP 搜索引荐。

    此变量可用于覆盖默认的 OpenLDAP 引荐配置；请参阅 LDAP 可插拔认证和 ldap.conf

+   `authentication_ldap_sasl_server_host`

    | 命令行格式 | `--authentication-ldap-sasl-server-host=host_name` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_server_host` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    对于 SASL LDAP 认证，LDAP 服务器主机。此变量的允许值取决于认证方法：

    +   对于`authentication_ldap_sasl_auth_method_name=SCRAM-SHA-1`：LDAP 服务器主机可以是主机名或 IP 地址。

    +   对于`authentication_ldap_sasl_auth_method_name=SCRAM-SHA-256`：LDAP 服务器主机可以是主机名或 IP 地址。

+   `authentication_ldap_sasl_server_port`

    | 命令行格式 | `--authentication-ldap-sasl-server-port=port_num` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_server_port` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `389` |
    | 最小值 | `1` |
    | 最大值 | `32376` |

    对于 SASL LDAP 认证，LDAP 服务器的 TCP/IP 端口号。

    从 MySQL 8.0.14 开始，如果 LDAP 端口号配置为 636 或 3269，则插件将使用 LDAPS（SSL 上的 LDAP）而不是 LDAP。（LDAPS 与`startTLS`不同。）

+   `authentication_ldap_sasl_tls`

    | 命令行格式 | `--authentication-ldap-sasl-tls[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_tls` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    对于 SASL LDAP 认证，插件与 LDAP 服务器的连接是否安全。如果启用此变量，插件将使用 TLS 安全连接到 LDAP 服务器。此变量可用于覆盖默认的 OpenLDAP TLS 配置；请参阅 LDAP 可插拔认证和 ldap.conf。如果您启用此变量，可能还希望设置`authentication_ldap_sasl_ca_path`变量。

    MySQL LDAP 插件支持 StartTLS 方法，该方法在普通 LDAP 连接的顶部初始化 TLS。

    截至 MySQL 8.0.14，可以通过设置`authentication_ldap_sasl_server_port`系统变量来使用 LDAPS。

+   `authentication_ldap_sasl_user_search_attr`

    | 命令行格式 | `--authentication-ldap-sasl-user-search-attr=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_sasl_user_search_attr` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `uid` |

    对于 SASL LDAP 身份验证，指定 LDAP 目录条目中的用户名的属性名称。如果未提供用户可分辨名称，身份验证插件将使用此属性搜索名称。例如，如果`authentication_ldap_sasl_user_search_attr`的值为`uid`，则搜索用户名称`user1`会找到具有`uid`值为`user1`的条目。

+   `authentication_ldap_simple_auth_method_name`

    | 命令行格式 | `--authentication-ldap-simple-auth-method-name=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_auth_method_name` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `SIMPLE` |
    | 有效值 | `SIMPLE``AD-FOREST` |

    对于简单的 LDAP 身份验证，身份验证方法名称。身份验证插件与 LDAP 服务器之间的通信根据此身份验证方法进行。

    注意

    对于所有简单的 LDAP 身份验证方法，建议还设置 TLS 参数，要求与 LDAP 服务器的通信必须通过安全连接进行。

    允许使用这些身份验证方法值：

    +   `SIMPLE`: 使用简单的 LDAP 身份验证。该方法使用一个或两个 LDAP 绑定操作，具体取决于 MySQL 账户是否命名了 LDAP 用户的可分辨名称。请参阅`authentication_ldap_simple_bind_root_dn`的描述。

    +   `AD-FOREST`: 一种基于`SIMPLE`的变体，使得身份验证在 Active Directory forest 中搜索所有域，在每个 Active Directory 域上执行 LDAP 绑定，直到在某个域中找到用户。

+   `authentication_ldap_simple_bind_base_dn`

    | 命令行格式 | `--authentication-ldap-simple-bind-base-dn=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_bind_base_dn` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于简单的 LDAP 身份验证，基本分明名称（DN）。此变量可用于通过在搜索树中的特定位置（“基本”）锚定它们来限制搜索范围。

    假设一组 LDAP 用户条目的成员各自具有以下形式：

    ```sql
    uid=*user_name*,ou=People,dc=example,dc=com
    ```

    另一组 LDAP 用户条目的成员具有以下形式：

    ```sql
    uid=*user_name*,ou=Admin,dc=example,dc=com
    ```

    然后，不同基本 DN 值的搜索工作如下：

    +   如果基本 DN 为`ou=People,dc=example,dc=com`：搜索仅在第一组中找到用户条目。

    +   如果基本 DN 为`ou=Admin,dc=example,dc=com`：搜索仅在第二组中找到用户条目。

    +   如果基本 DN 为`ou=dc=example,dc=com`：搜索在第一组或第二组中找到用户条目。

    一般来说，更具体的基本 DN 值会导致更快的搜索，因为它们限制了搜索范围。

+   `authentication_ldap_simple_bind_root_dn`

    | 命令行格式 | `--authentication-ldap-simple-bind-root-dn=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_bind_root_dn` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于简单的 LDAP 身份验证，根分明名称（DN）。此变量与`authentication_ldap_simple_bind_root_pwd`一起用作验证到 LDAP 服务器以执行搜索的凭据。身份验证使用一个或两个 LDAP 绑定操作，具体取决于 MySQL 帐户是否命名为 LDAP 用户 DN：

    +   如果帐户没有指定用户 DN：`authentication_ldap_simple`使用`authentication_ldap_simple_bind_root_dn`和`authentication_ldap_simple_bind_root_pwd`执行初始 LDAP 绑定。（这两个变量默认为空，因此如果它们未设置，则 LDAP 服务器必须允许匿名连接。）生成的绑定 LDAP 句柄用于根据客户端用户名搜索用户 DN。`authentication_ldap_simple`使用用户 DN 和客户端提供的密码执行第二次绑定。

    +   如果帐户没有指定用户 DN：在这种情况下，第一个绑定操作是不必要的。`authentication_ldap_simple`使用用户 DN 和客户端提供的密码执行单个绑定。这比如果 MySQL 帐户没有指定 LDAP 用户 DN 要快。

+   `authentication_ldap_simple_bind_root_pwd`

    | 命令行格式 | `--authentication-ldap-simple-bind-root-pwd=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_bind_root_pwd` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于简单的 LDAP 身份验证，根分隔名称的密码。此变量与`authentication_ldap_simple_bind_root_dn`一起使用。请参阅该变量的描述。

+   `authentication_ldap_simple_ca_path`

    | 命令行格式 | `--authentication-ldap-simple-ca-path=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_ca_path` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    对于简单的 LDAP 身份验证，证书颁发机构文件的绝对路径。如果希望认证插件执行 LDAP 服务器证书的验证，请指定此文件。

    注意

    除了将`authentication_ldap_simple_ca_path`变量设置为文件名之外，还必须将适当的证书颁发机构证书添加到文件中，并启用`authentication_ldap_simple_tls`系统变量。这些变量可以设置以覆盖默认的 OpenLDAP TLS 配置；请参阅 LDAP Pluggable Authentication and ldap.conf

+   `authentication_ldap_simple_group_search_attr`

    | 命令行格式 | `--authentication-ldap-simple-group-search-attr=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_group_search_attr` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `cn` |

    对于简单的 LDAP 认证，指定 LDAP 目录条目中组名的属性名称。如果`authentication_ldap_simple_group_search_attr`具有其默认值`cn`，搜索将返回`cn`值作为组名。例如，如果具有`uid`值为`user1`的 LDAP 条目具有`cn`属性为`mygroup`，则搜索`user1`将返回`mygroup`作为组名。

    如果组搜索属性是`isMemberOf`，LDAP 认证直接检索用户属性`isMemberOf`的值，并将其分配为组信息。如果组搜索属性不是`isMemberOf`，LDAP 认证将搜索用户是成员的所有组。（后者是默认行为。）此行为基于 LDAP 组信息可以以两种方式存储的方式：1）组条目可以具有名为`memberUid`或`member`的属性，其值为用户名；2）用户条目可以具有名为`isMemberOf`的属性，其值为组名。

+   `authentication_ldap_simple_group_search_filter`

    | 命令行格式 | `--authentication-ldap-simple-group-search-filter=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_group_search_filter` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `(&#124;(&(objectClass=posixGroup)(memberUid=%s))(&(objectClass=group)(member=%s)))` |

    对于简单的 LDAP 认证，自定义组搜索过滤器。

    搜索过滤器值可以包含`{UA}`和`{UD}`符号，分别表示用户名和完整用户 DN。例如，`{UA}`被替换为用户名，如`"admin"`，而`{UD}`被替换为完整 DN，如`"uid=admin,ou=People,dc=example,dc=com"`。以下值是默认值，支持 OpenLDAP 和 Active Directory：

    ```sql
    (|(&(objectClass=posixGroup)(memberUid={UA}))
      (&(objectClass=group)(member={UD})))
    ```

    在某些用户场景中，`memberOf`是一个简单的用户属性，不包含任何组信息。为了增加灵活性，可以在组搜索属性中使用可选的`{GA}`前缀。任何带有{GA}前缀的组属性都被视为具有组名的用户属性。例如，如果值为`{GA}MemberOf`，如果组值是 DN，则从组 DN 中返回第一个属性值作为组名。

+   `authentication_ldap_simple_init_pool_size`

    | 命令行格式 | `--authentication-ldap-simple-init-pool-size=#` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_init_pool_size` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10` |
    | 最小值 | `0` |
    | 最大值 | `32767` |
    | 单位 | 连接 |

    对于简单的 LDAP 认证，连接到 LDAP 服务器的连接池的初始大小。根据对 LDAP 服务器的平均并发认证请求数量选择此变量的值。

    插件同时使用`authentication_ldap_simple_init_pool_size`和`authentication_ldap_simple_max_pool_size`进行连接池管理：

    +   当认证插件初始化时，它会创建`authentication_ldap_simple_init_pool_size`个连接，除非`authentication_ldap_simple_max_pool_size=0`以禁用连接池。

    +   如果插件在当前连接池中没有空闲连接时收到认证请求，则插件可以创建一个新连接，最多达到`authentication_ldap_simple_max_pool_size`给定的最大连接池大小。

    +   如果插件在池大小已达到最大值且没有空闲连接时收到请求，则身份验证失败。

    +   当插件卸载时，它会关闭所有连接池中的连接。

    对插件系统变量设置的更改可能对已经在池中的连接没有影响。例如，修改 LDAP 服务器主机、端口或 TLS 设置不会影响现有连接。但是，如果原始变量值无效且连接池无法初始化，则插件会尝试为下一个 LDAP 请求重新初始化池。在这种情况下，新的系统变量值将用于重新初始化尝试。

    如果`authentication_ldap_simple_max_pool_size=0`以禁用连接池，则插件打开的每个 LDAP 连接都使用系统变量在那时的值。

+   `authentication_ldap_simple_log_status`

    | 命令行格式 | `--authentication-ldap-simple-log-status=#` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_log_status` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值（≥ 8.0.18） | `6` |
    | 最大值（≤ 8.0.17） | `5` |

    对于简单的 LDAP 身份验证，写入错误日志的消息的日志级别。下表显示了允许的级别值及其含义。

    **表 8.31 authentication_ldap_simple_log_status 的日志级别**

    | 选项数值 | 记录的消息类型 |
    | --- | --- |
    | `1` | 无消息 |
    | `2` | 错误消息 |
    | `3` | 错误和警告消息 |
    | `4` | 错误、警告和信息消息 |
    | `5` | 与前一级别相同，加上来自 MySQL 的调试消息 |
    | `6` | 与前一级别相同，加上来自 LDAP 库的调试消息 |

    MySQL 8.0.18 版本中提供日志级别 6。

+   `authentication_ldap_simple_max_pool_size`

    | 命令行格式 | `--authentication-ldap-simple-max-pool-size=#` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_max_pool_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1000` |
    | 最小值 | `0` |
    | 最大值 | `32767` |
    | 单位 | 连接数 |

    对于简单的 LDAP 身份��证，连接到 LDAP 服务器的连接池的最大大小。要禁用连接池，请将此变量设置为 0。

    此变量与 `authentication_ldap_simple_init_pool_size` 结合使用。请参阅该变量的描述。

+   `authentication_ldap_simple_referral`

    | 命令行格式 | `--authentication-ldap-simple-referral[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `authentication_ldap_simple_referral` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    对于简单的 LDAP 认证，是否启用 LDAP 搜索引荐。请参阅 LDAP 搜索引荐。

+   `authentication_ldap_simple_server_host`

    | 命令行格式 | `--authentication-ldap-simple-server-host=host_name` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_server_host` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    对于简单的 LDAP 认证，LDAP 服务器主机。此变量的允许值取决于认证方法：

    +   对于 `authentication_ldap_simple_auth_method_name=SIMPLE`: LDAP 服务器主机可以是主机名或 IP 地址。

    +   对于 `authentication_ldap_simple_auth_method_name=AD-FOREST`。LDAP 服务器主机可以是 Active Directory 域名。例如，对于 LDAP 服务器 URL 为 `ldap://example.mem.local:389`，域名可以是 `mem.local`。

        Active Directory 森林设置可以拥有多个域（LDAP 服务器 IP），可以使用 DNS 发现。在 Unix 和类 Unix 系统上，可能需要进行一些额外的设置来配置您的 DNS 服务器，使用指定 Active Directory 域的 LDAP 服务器的 SRV 记录。有关 DNS SRV 的信息，请参阅 [RFC 2782](https://tools.ietf.org/html/rfc2782)。

        假设您的配置具有以下属性：

        +   提供有关 Active Directory 域信息的名称服务器的 IP 地址为 `10.172.166.100`。

        +   LDAP 服务器的名称为 `ldap1.mem.local` 到 `ldap3.mem.local`，IP 地址为 `10.172.166.101` 到 `10.172.166.103`。

        希望能够通过 SRV 搜索发现 LDAP 服务器。例如，在命令行中，类似这样的命令应该列出 LDAP 服务器：

        ```sql
        host -t SRV _ldap._tcp.mem.local
        ```

        执行以下 DNS 配置：

        1.  添加一行到 `/etc/resolv.conf`，指定提供有关 Active Directory 域信息的名称服务器：

            ```sql
            nameserver 10.172.166.100
            ```

        1.  针对 LDAP 服务器配置适当的区域文件，包含 LDAP 服务器的 SRV 记录：

            ```sql
            _ldap._tcp.mem.local. 86400 IN SRV 0 100 389 ldap1.mem.local.
            _ldap._tcp.mem.local. 86400 IN SRV 0 100 389 ldap2.mem.local.
            _ldap._tcp.mem.local. 86400 IN SRV 0 100 389 ldap3.mem.local.
            ```

        1.  如果服务器主机无法解析，可能还需要在 `/etc/hosts` 中指定 LDAP 服务器的 IP 地址。例如，向文件中添加如下行：

            ```sql
            10.172.166.101 ldap1.mem.local
            10.172.166.102 ldap2.mem.local
            10.172.166.103 ldap3.mem.local
            ```

        配置 DNS 如刚才描述的那样，服务器端 LDAP 插件可以发现 LDAP 服务器，并在所有域中尝试进行身份验证，直到身份验证成功或没有更多服务器为止。

        Windows 不需要像刚才描述的设置。给定 `authentication_ldap_simple_server_host` 值中的 LDAP 服务器主机，Windows LDAP 库会搜索所有域并尝试进行身份验证。

+   `authentication_ldap_simple_server_port`

    | 命令行格式 | `--authentication-ldap-simple-server-port=port_num` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_server_port` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `389` |
    | 最小值 | `1` |
    | 最大值 | `32376` |

    简单 LDAP 身份验证所需的 LDAP 服务器 TCP/IP 端口号。

    从 MySQL 8.0.14 开始，如果 LDAP 端口号配置为 636 或 3269，则插件将使用 LDAPS（LDAP over SSL）而不是 LDAP。（LDAPS 与 `startTLS` 不同。）

+   `authentication_ldap_simple_tls`

    | 命令行格式 | `--authentication-ldap-simple-tls[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_tls` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    对于简单的 LDAP 认证，插件与 LDAP 服务器的连接是否安全。如果启用此变量，插件将使用 TLS 安全连接到 LDAP 服务器。此变量可用于覆盖默认的 OpenLDAP TLS 配置；请参阅 LDAP 可插拔认证和 ldap.conf。如果启用此变量，您可能还希望设置 `authentication_ldap_simple_ca_path` 变量。

    MySQL LDAP 插件支持 StartTLS 方法，该方法在普通 LDAP 连接的基础上初始化 TLS。

    从 MySQL 8.0.14 开始，可以通过设置 `authentication_ldap_simple_server_port` 系统变量来使用 LDAPS。

+   `authentication_ldap_simple_user_search_attr`

    | 命令行格式 | `--authentication-ldap-simple-user-search-attr=value` |
    | --- | --- |
    | 系统变量 | `authentication_ldap_simple_user_search_attr` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `uid` |

    对于简单的 LDAP 认证，指定 LDAP 目录条目中用户名称的属性名称。如果未提供用户的可分辨名称，认证插件将使用此属性搜索名称。例如，如果 `authentication_ldap_simple_user_search_attr` 的值为 `uid`，则搜索用户名称 `user1` 将找到具有 `uid` 值为 `user1` 的条目。
