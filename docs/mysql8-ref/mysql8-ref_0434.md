> 原文：[`dev.mysql.com/doc/refman/8.0/en/keyring-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/keyring-system-variables.html)

#### 8.4.4.19 密钥环系统变量

MySQL 密钥环插件支持以下系统变量。使用它们来配置密钥环插件操作。除非安装了适当的密钥环插件（请参阅第 8.4.4.3 节，“密钥环插件安装”），否则这些变量不可用。

+   `keyring_aws_cmk_id`

    | 命令行格式 | `--keyring-aws-cmk-id=value` |
    | --- | --- |
    | 系统变量 | `keyring_aws_cmk_id` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    从 AWS KMS 服务器获取并由`keyring_aws` 插件使用的 KMS 密钥 ID。除非安装了该插件，否则此变量不可用。

    此变量为必填项。如果未指定，`keyring_aws` 初始化将失败。

+   `keyring_aws_conf_file`

    | 命令行格式 | `--keyring-aws-conf-file=file_name` |
    | --- | --- |
    | 系统变量 | `keyring_aws_conf_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `特定于平台` |

    `keyring_aws` 插件的配置文件位置。除非安装了该插件，否则此变量不可用。

    在插件启动时，`keyring_aws` 从配置文件中读取 AWS 秘密访问密钥 ID 和密钥。为了使`keyring_aws` 插件成功启动，配置文件必须存在并包含有效的秘密访问密钥信息，初始化如第 8.4.4.9 节，“使用 keyring_aws 亚马逊网络服务密钥环插件”中所述。

    默认文件名为`keyring_aws_conf`，位于默认密钥环文件目录中。此默认目录的位置与`keyring_file_data`系统变量相同。有关详细信息，请参阅该变量的描述，以及如果手动创建目录时要考虑的注意事项。

+   `keyring_aws_data_file`

    | 命令行格式 | `--keyring-aws-data-file` |
    | --- | --- |
    | 系统变量 | `keyring_aws_data_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `特定于平台` |

    存储`keyring_aws`插件的文件位置。除非安装了该插件，否则此变量不可用。

    在插件启动时，如果分配给`keyring_aws_data_file`的值指定不存在的文件，则`keyring_aws`插件会尝试创建它（以及必要时的父目录）。如果文件存在，`keyring_aws`会将文件中包含的任何加密密钥读入其内存缓存中。`keyring_aws`不会在内存中缓存未加密的密钥。

    默认文件名为`keyring_aws_data`，位于默认密钥环文件目录中。此默认目录的位置与`keyring_file_data`系统变量相同。有关详细信息，请参阅该变量的描述，以及如果手动创建目录时需要考虑的事项。

+   `keyring_aws_region`

    | 命令行格式 | `--keyring-aws-region=value` |
    | --- | --- |
    | 系统变量 | `keyring_aws_region` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `us-east-1` |
    | 有效值（≥ 8.0.30） | `af-south-1``ap-east-1``ap-northeast-1``ap-northeast-2``ap-northeast-3``ap-south-1``ap-southeast-1``ap-southeast-2``ca-central-1``cn-north-1``cn-northwest-1``eu-central-1``eu-north-1``eu-south-1``eu-west-1``eu-west-2``eu-west-3``me-south-1``sa-east-1``us-east-1``us-east-2``us-gov-east-1``us-iso-east-1``us-iso-west-1``us-isob-east-1``us-west-1``us-west-2` |
    | 有效值（≥ 8.0.17, ≤ 8.0.29） | `ap-northeast-1``ap-northeast-2``ap-south-1``ap-southeast-1``ap-southeast-2``ca-central-1``cn-north-1``cn-northwest-1``eu-central-1``eu-west-1``eu-west-2``eu-west-3``sa-east-1``us-east-1``us-east-2``us-west-1``us-west-2` |
    | 有效值（≤ 8.0.16） | `ap-northeast-1``ap-northeast-2``ap-south-1``ap-southeast-1``ap-southeast-2``eu-central-1``eu-west-1``sa-east-1``us-east-1``us-west-1``us-west-2` |

    `keyring_aws`插件的 AWS 区域。除非安装了该插件，否则此变量不可用。

+   `keyring_encrypted_file_data`

    | 命令行格式 | `--keyring-encrypted-file-data=file_name` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `keyring_encrypted_file_data` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `特定于平台` |

    注意

    截至 MySQL 8.0.34 版本，`keyring_encrypted_file` 插件已被弃用，并可能在将来的 MySQL 版本中被移除。考虑使用 `component_keyring_encrypted_file` 替代；`component_keyring_encrypted_file` 组件取代了 `keyring_encrypted_file` 插件。

    由 `keyring_encrypted_file` 插件用于安全数据存储的数据文件的路径名。除非安装了该插件，否则此变量不可用。文件位置应位于仅供密钥环插件使用的目录中。例如，不要将文件放在数据目录下。

    密钥环操作是事务性的：`keyring_encrypted_file` 插件在写操作期间使用备份文件，以确保如果操作失败，可以回滚到原始文件。备份文件与 `keyring_encrypted_file_data` 系统变量的值相同，后缀为 `.backup`。

    不要为多个 MySQL 实例使用相同的 `keyring_encrypted_file` 数据文件。每个实例应该有自己独特的数据文件。

    默认文件名为 `keyring_encrypted`，位于特定于平台的目录中，取决于 `INSTALL_LAYOUT` **CMake** 选项的值，如下表所示。如果您正在从源代码构建，并且要明确指定文件的默认目录，请使用 `INSTALL_MYSQLKEYRINGDIR` **CMake** 选项。

    | `INSTALL_LAYOUT` 值 | 默认 `keyring_encrypted_file_data` 值 |
    | --- | --- |
    | `DEB`, `RPM`, `SVR4` | `/var/lib/mysql-keyring/keyring_encrypted` |
    | 否则 | 在 `CMAKE_INSTALL_PREFIX` 值下的 `keyring/keyring_encrypted` |

    在插件启动时，如果分配给 `keyring_encrypted_file_data` 的值指定一个不存在的文件，则 `keyring_encrypted_file` 插件会尝试创建它（以及必要时的父目录）。

    如果您手动创建目录，应该具有严格的模式，并且只能由用于运行 MySQL 服务器的帐户访问。例如，在 Unix 和类 Unix 系统上，要使用 `/usr/local/mysql/mysql-keyring` 目录，以下命令（以 `root` 执行）创建目录并设置其模式和所有权：

    ```sql
    cd /usr/local/mysql
    mkdir mysql-keyring
    chmod 750 mysql-keyring
    chown mysql mysql-keyring
    chgrp mysql mysql-keyring
    ```

    如果 `keyring_encrypted_file` 插件无法创建或访问其数据文件，它会将错误消息写入错误日志。如果尝试对 `keyring_encrypted_file_data` 进行运行时赋值导致错误，则变量值保持不变。

    重要

    一旦 `keyring_encrypted_file` 插件创建了其数据文件并开始使用它，重要的是不要删除该文件。文件丢失会导致使用其密钥加密的数据无法访问。（可以重命名或移动文件，只要您将 `keyring_encrypted_file_data` 的值相匹配即可。）

+   `keyring_encrypted_file_password`

    | 命令行格式 | `--keyring-encrypted-file-password=password` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `keyring_encrypted_file_password` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    注意

    截至 MySQL 8.0.34 版本，`keyring_encrypted_file` 插件已被弃用，并可能在未来的 MySQL 版本中被移除。考虑使用 `component_keyring_encrypted_file` 替代；`component_keyring_encrypted_file` 组件取代了 `keyring_encrypted_file` 插件。

    `keyring_encrypted_file` 插件使用的密码。除非安装了该插件，否则此变量不可用。

    此变量是强制性的。如果未指定，`keyring_encrypted_file` 初始化将失败。

    如果在选项文件中指定了此变量，则文件应具有限制模式，并且只能由用于运行 MySQL 服务器的帐户访问。

    重要

    一旦设置了 `keyring_encrypted_file_password` 的值，更改它不会旋转密钥环密码，并且可能导致服务器无法访问。如果提供了错误密码，则 `keyring_encrypted_file` 插件无法从加密的密钥环文件中加载密钥。

    密码数值无法在运行时通过 `SHOW VARIABLES` 或性能模式 `global_variables` 表显示，因为显示数值已被混淆。

+   `keyring_file_data`

    | 命令行格式 | `--keyring-file-data=file_name` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `keyring_file_data` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `特定于平台` |

    注意

    截至 MySQL 8.0.34 版本，`keyring_file` 插件已被弃用，并可能在未来的 MySQL 版本中被移除。考虑使用 `component_keyring_file` 替代；`component_keyring_file` 组件取代了 `keyring_file` 插件。

    由 `keyring_file` 插件用于安全数据存储的数据文件的路径名。除非安装了该插件，否则此变量不可用。文件位置应位于仅供密钥环插件使用的目录中。例如，不要将文件放在数据目录下。

    密钥环操作是事务性的：`keyring_file` 插件在写操作期间使用备份文件，以确保如果操作失败，它可以回滚到原始文件。备份文件与 `keyring_file_data` 系统变量的值相同，后缀为 `.backup`。

    不要为多个 MySQL 实例使用相同的 `keyring_file` 数据文件。每个实例应具有其自己独特的数据文件。

    默认文件名为 `keyring`，位于特定于平台的目录中，并取决于 `INSTALL_LAYOUT` **CMake** 选项的值，如下表所示。如果您正在从源代码构建，则可以使用 `INSTALL_MYSQLKEYRINGDIR` **CMake** 选项显式指定文件的默认目录。

    | `INSTALL_LAYOUT` 值 | 默认 `keyring_file_data` 值 |
    | --- | --- |
    | `DEB`, `RPM`, `SVR4` | `/var/lib/mysql-keyring/keyring` |
    | 否则 | 在 `CMAKE_INSTALL_PREFIX` 值下的 `keyring/keyring` |

    在插件启动时，如果分配给 `keyring_file_data` 的值指定一个不存在的文件，则 `keyring_file` 插件会尝试创建它（以及必要时的父目录）。

    如果您手动创建目录，则应具有限制模式，并且只能由用于运行 MySQL 服务器的帐户访问。例如，在 Unix 和类 Unix 系统上，要使用 `/usr/local/mysql/mysql-keyring` 目录，以下命令（以 `root` 执行）创建目录并设置其模式和所有权：

    ```sql
    cd /usr/local/mysql
    mkdir mysql-keyring
    chmod 750 mysql-keyring
    chown mysql mysql-keyring
    chgrp mysql mysql-keyring
    ```

    如果 `keyring_file` 插件无法创建或访问其数据文件，则会将错误消息写入错误日志。如果对 `keyring_file_data` 的尝试运行时赋值导致错误，则变量值保持不变。

    重要

    一旦`keyring_file`插件创建了其数据文件并开始使用它，重要的是不要删除该文件。例如，`InnoDB`使用该文件存储用于解密使用`InnoDB`表空间加密的数据的主密钥；请参阅第 17.13 节，“InnoDB 数据静态加密”。文件丢失会导致这些表中的数据无法访问。（可以重命名或移动文件，只要您更改`keyring_file_data`的值以匹配即可。）建议在创建第一个加密表之后立即创建密钥环数据文件的单独备份，并在主密钥轮换之前和之后创建备份。

+   `keyring_hashicorp_auth_path`

    | 命令行格式 | `--keyring-hashicorp-auth-path=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_auth_path` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认�� | `/v1/auth/approle/login` |

    在 HashiCorp Vault 服务器中启用 AppRole 认证的认证路径，供`keyring_hashicorp`插件使用。除非安装了该插件，否则此变量不可用。

+   `keyring_hashicorp_ca_path`

    | 命令行格式 | `--keyring-hashicorp-ca-path=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_ca_path` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `空字符串` |

    MySQL 服务器可访问的本地文件的绝对路径名，其中包含供`keyring_hashicorp`插件使用的正确格式化的 TLS 证书颁发机构。除非安装了该插件，否则此变量不可用。

    如果未设置此变量，`keyring_hashicorp`插件将在不使用服务器证书验证的情况下打开 HTTPS 连接，并信任 HashiCorp Vault 服务器提供的任何证书。为了安全起见，必须假定 Vault 服务器不是恶意的，且不存在中间人攻击的可能性。如果这些假设是无效的，请将`keyring_hashicorp_ca_path`设置为受信任 CA 证书的路径。（例如，在证书和密钥准备中的说明中，这是`company.crt`文件。）

+   `keyring_hashicorp_caching`

    | 命令行格式 | `--keyring-hashicorp-caching[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_caching` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    是否启用`keyring_hashicorp`插件使用的可选内存中密钥缓存，以缓存来自 HashiCorp Vault 服务器的密钥。除非安装了该插件，否则此变量不可用。如果启用了缓存，插件将在初始化期间填充它。否则，插件仅在初始化期间填充密钥列表。

    启用缓存是一种折衷：它提高了性能，但在内存中保留了敏感密钥信息的副本，这可能不利于安全目的。

+   `keyring_hashicorp_commit_auth_path`

    | 引入 | 8.0.18 |
    | --- | --- |
    | 系统变量 | `keyring_hashicorp_commit_auth_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量与`keyring_hashicorp_auth_path`相关联，在`keyring_hashicorp`插件初始化期间从中获取其值。除非安装了该插件，否则此变量不可用。如果初始化成功，它反映了实际用于插件操作的“已提交”值。有关更多信息，请参见 keyring_hashicorp 配置。

+   `keyring_hashicorp_commit_ca_path`

    | 引入 | 8.0.18 |
    | --- | --- |
    | 系统变量 | `keyring_hashicorp_commit_ca_path` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量与`keyring_hashicorp_ca_path`相关联，在`keyring_hashicorp`插件初始化期间从中获取其值。除非安装了该插件，否则此变量不可用。如果初始化成功，它反映了实际用于插件操作的“已提交”值。有关更多信息，请参见 keyring_hashicorp 配置。

+   `keyring_hashicorp_commit_caching`

    | 引入版本 | 8.0.18 |
    | --- | --- |
    | 系统变量 | `keyring_hashicorp_commit_caching` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量与`keyring_hashicorp_caching`相关联，在`keyring_hashicorp`插件初始化期间从中获取其值。除非安装了该插件，否则此变量不可用。如果初始化成功，它反映了实际用于插件操作的“已提交”值。有关更多信息，请参见 keyring_hashicorp 配置。

+   `keyring_hashicorp_commit_role_id`

    | 引入版本 | 8.0.18 |
    | --- | --- |
    | 系统变量 | `keyring_hashicorp_commit_role_id` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量与`keyring_hashicorp_role_id`相关联，在`keyring_hashicorp`插件初始化期间从中获取其值。除非安装了该插件，否则此变量不可用。如果初始化成功，它反映了实际用于插件操作的“已提交”值。有关更多信息，请参见 keyring_hashicorp 配置。

+   `keyring_hashicorp_commit_server_url`

    | 引入版本 | 8.0.18 |
    | --- | --- |
    | 系统变量 | `keyring_hashicorp_commit_server_url` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量与`keyring_hashicorp_server_url`相关联，在`keyring_hashicorp`插件初始化期间从中获取其值。除非安装了该插件，否则此变量不可用。如果初始化成功，它反映了实际用于插件操作的“已提交”值。有关更多信息，请参见 keyring_hashicorp 配置。

+   `keyring_hashicorp_commit_store_path`

    | 引入版本 | 8.0.18 |
    | --- | --- |
    | 系统变量 | `keyring_hashicorp_commit_store_path` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    此变量与`keyring_hashicorp_store_path`相关联，在`keyring_hashicorp` 插件初始化期间从中获取其值。除非安装了该插件，否则此变量不可用。如果初始化成功，它反映了实际用于插件操作的“已提交”值。有关更多信息，请参阅 keyring_hashicorp 配置。

+   `keyring_hashicorp_role_id`

    | 命令行格式 | `--keyring-hashicorp-role-id=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_role_id` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    HashiCorp Vault AppRole 认证角色 ID，供`keyring_hashicorp` 插件使用。除非安装了该插件，否则此变量不可用。值必须采用 UUID 格式。

    此变量为必填项。如果未指定，`keyring_hashicorp` 初始化将失败。

+   `keyring_hashicorp_secret_id`

    | 命令行格式 | `--keyring-hashicorp-secret-id=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_secret_id` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    HashiCorp Vault AppRole 认证秘钥 ID，供`keyring_hashicorp` 插件使用。除非安装了该插件，否则此变量不可用。值必须采用 UUID 格式。

    此变量为必填项。如果未指定，`keyring_hashicorp` 初始化将失败。

    此变量的值属于敏感信息，因此在显示时会被`*`字符掩盖。

+   `keyring_hashicorp_server_url`

    | 命令行格式 | `--keyring-hashicorp-server-url=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_server_url` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `https://127.0.0.1:8200` |

    HashiCorp Vault 服务器 URL，供 `keyring_hashicorp` ��件使用。除非安装了该插件，否则此变量不可用。值必须以 `https://` 开头。

+   `keyring_hashicorp_store_path`

    | 命令行格式 | `--keyring-hashicorp-store-path=value` |
    | --- | --- |
    | 引入版本 | 8.0.18 |
    | 系统变量 | `keyring_hashicorp_store_path` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    HashiCorp Vault 服务器内的存储路径，在适当提供 AppRole 凭据的情况下由 `keyring_hashicorp` 插件提供写入权限。除非安装了该插件，否则此变量不可用。要指定凭据，请设置 `keyring_hashicorp_role_id` 和 `keyring_hashicorp_secret_id` 系统变量（例如，在 keyring_hashicorp 配置 中所示）。

    此变量为必填项。如果未指定，`keyring_hashicorp` 初始化将失败。

+   `keyring_oci_ca_certificate`

    | 命令行格式 | `--keyring-oci-ca-certificate=file_name` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_ca_certificate` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `空字符串` |

    用于 Oracle Cloud Infrastructure 证书验证的 CA 证书包文件的路径名。除非安装了该插件，否则此变量不可用。

    文件包含一个或多个用于对等验证的证书。如果未指定文件，则使用系统上安装的默认 CA 包。如果值为 `disabled`（区分大小写），`keyring_oci` 将不执行证书验证。

+   `keyring_oci_compartment`

    | 命令行格式 | `--keyring-oci-compartment=ocid` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_compartment` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    `keyring_oci` 插件用作 MySQL 密钥位置的租户区段的 OCID。除非安装了该插件，否则此变量不可用。

    在使用 `keyring_oci` 之前，如果不存在 MySQL 区段或子区段，则必须创建一个。此区段不应包含任何保险库密钥或保险库密钥。它不应被 MySQL Keyring 以外的系统使用。

    有关管理区段和获取 OCID 的信息，请参阅[管理区段](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcompartments.htm)。

    此变量是必需的。如果未指定，`keyring_oci` 初始化将失败。

+   `keyring_oci_encryption_endpoint`

    | 命令行格式 | `--keyring-oci-encryption-endpoint=value` |
    | --- | --- |
    | 引入 | 8.0.22 |
    | 弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_encryption_endpoint` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    Oracle Cloud Infrastructure 加密服务器的端点，`keyring_oci` 插件用于为新密钥生成密文。除非安装了该插件，否则此变量不可用。

    加密端点是特定于保险库的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取端点 OCID，请查看您的 `keyring_oci` 保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

    此变量是必需的。如果未指定，`keyring_oci` 初始化将失败。

+   `keyring_oci_key_file`

    | 命令行格式 | `--keyring-oci-key-file=file_name` |
    | --- | --- |
    | 引入 | 8.0.22 |
    | 弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_key_file` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    包含 `keyring_oci` 插件用于 Oracle Cloud Infrastructure 认证的 RSA 私钥文件的文件路径名。除非安装了该插件，否则此变量不可用。

    您还必须使用控制台上传相应的 RSA 公钥。控制台显示密钥指纹值，您可以使用它来设置`keyring_oci_key_fingerprint` 系统变量。

    有关生成和上传 API 密钥的信息，请参阅[必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)。

    此变量是强制性的。如果未指定，`keyring_oci` 初始化将失败。

+   `keyring_oci_key_fingerprint`

    | 命令行格式 | `--keyring-oci-key-fingerprint=value` |
    | --- | --- |
    | 引入 | 8.0.22 |
    | 弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_key_fingerprint` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    RSA 私钥的指纹，`keyring_oci` 插件用于 Oracle Cloud Infrastructure 认证。除非安装了该插件，否则此变量不可用。

    在创建 API 密钥时获取密钥指纹，请执行此命令：

    ```sql
    openssl rsa -pubout -outform DER -in ~/.oci/oci_api_key.pem | openssl md5 -c
    ```

    或者，从控制台获取指纹，当您上传 RSA 公钥时，控制台会自动显示指纹。

    有关获取密钥指纹的信息，请参阅[必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)。

    此变量是强制性的。如果未指定，`keyring_oci` 初始化将失败。

+   `keyring_oci_management_endpoint`

    | 命令行格式 | `--keyring-oci-management-endpoint=value` |
    | --- | --- |
    | 引入 | 8.0.22 |
    | 弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_management_endpoint` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    Oracle Cloud Infrastructure 密钥管理服务器的端点，`keyring_oci` 插件用于列出现有密钥。除非安装了该插件，否则此变量不可用。

    密钥管理端点是特定于保险库的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取端点 OCID，请查看您的`keyring_oci`保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

    此变量是强制性的。如果未指定，`keyring_oci` 初始化将失败。

+   `keyring_oci_master_key`

    | 命令行格式 | `--keyring-oci-master-key=ocid` |
    | --- | --- |
    | 引入 | 8.0.22 |
    | 弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_master_key` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |

    `keyring_oci`插件用于加密秘密的 Oracle Cloud Infrastructure 主加密密钥的 OCID。除非安装了该插件，否则此变量不可用。

    在使用`keyring_oci`之前，如果不存在 Oracle Cloud Infrastructure 区隔的加密密钥，则必须创建一个。为生成的密钥提供一个特定于 MySQL 的名称，并且不要将其用于其他目的。

    有关密钥创建的信息，请参阅[管理密钥](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingkeys.htm)。

    此变量是强制性的。如果未指定，`keyring_oci`初始化将失败。

+   `keyring_oci_secrets_endpoint`

    | 命令行格式 | `--keyring-oci-secrets-endpoint=value` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_secrets_endpoint` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    Oracle Cloud Infrastructure 秘密服务器的终端点，`keyring_oci`插件用于列出、创建和撤销秘密。除非安装了该插件，否则此变量不可用。

    秘密终端点是特定于保险库的，Oracle Cloud Infrastructure 在创建保险库时分配它。要获取终端点 OCID，请查看您的`keyring_oci`保险库的配置详细信息，使用[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

    此变量是强制性的。如果未指定，`keyring_oci`初始化将失败。

+   `keyring_oci_tenancy`

    | 命令行格式 | `--keyring-oci-tenancy=ocid` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_tenancy` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    `keyring_oci`插件用作 MySQL 区隔的 Oracle Cloud Infrastructure 租户的 OCID。除非安装了该插件，否则此变量不可用。

    在使用`keyring_oci`之前，如果不存在租户，则必须创建一个租户。要从控制台获取租户 OCID，请使用[必需的密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)中的说明。

    此变量是强制性的。如果未指定，`keyring_oci`初始化将失败。

+   `keyring_oci_user`

    | 命令行格式 | `--keyring-oci-user=ocid` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_user` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    `keyring_oci`插件用于云连接的 Oracle Cloud Infrastructure 用户的 OCID。除非安装了该插件，否则此变量不可用。

    在使用`keyring_oci`之前，此用户必须存在并被授予使用配置的 Oracle Cloud Infrastructure 租户、区域和 Vault 资源的访问权限。

    要从控制台获取用户 OCID，请使用[必需密钥和 OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)中的说明。

    此变量是必需的。如果未指定，`keyring_oci`初始化将失败。

+   `keyring_oci_vaults_endpoint`

    | 命令行格式 | `--keyring-oci-vaults-endpoint=value` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_vaults_endpoint` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    `keyring_oci`插件用于获取密钥值的 Oracle Cloud Infrastructure Vaults 服务器的端点。除非安装了该插件，否则此变量不可用。

    Vault 端点是特定于 Vault 的，Oracle Cloud Infrastructure 在创建 Vault 时分配它。要获取端点 OCID，请查看您的`keyring_oci` Vault 的配置详细信息，使用[管理 Vaults](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)中的说明。

    此变量是必需的。如果未指定，`keyring_oci`初始化将失败。

+   `keyring_oci_virtual_vault`

    | 命令行格式 | `--keyring-oci-virtual-vault=ocid` |
    | --- | --- |
    | 引入版本 | 8.0.22 |
    | 已弃用 | 8.0.31 |
    | 系统变量 | `keyring_oci_virtual_vault` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |

    Oracle Cloud Infrastructure Vault 的 OCID，`keyring_oci`插件用于加密操作的 Vault。除非安装了该插件，否则此变量不可用。

    在使用`keyring_oci`之前，如果 MySQL 区域不存在，则必须在 MySQL 区域中创建一个新的 Vault。（或者，您可以重用位于 MySQL 区域的父区域中的现有 Vault。）区域用户只能看到并使用其各自区域中的密钥。

    有关创建保险库并获取保险库 OCID 的信息，请参阅[管理保险库](https://docs.cloud.oracle.com/en-us/iaas/Content/KeyManagement/Tasks/managingvaults.htm)。

    该变量是强制性的。如果未指定，`keyring_oci`初始化将失败。

+   `keyring_okv_conf_dir`

    | 命令行格式 | `--keyring-okv-conf-dir=dir_name` |
    | --- | --- |
    | 系统变量 | `keyring_okv_conf_dir` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 目录名称 |
    | 默认值 | `空字符串` |

    存储`keyring_okv`插件使用的配置信息的目录路径。除非安装了该插件，否则此变量不可用。该位置应该是仅供`keyring_okv`插件使用的目录。例如，不要将目录放在数据目录下。

    默认`keyring_okv_conf_dir`值为空。为了让`keyring_okv`插件能够访问 Oracle Key Vault，该值必须设置为包含 Oracle Key Vault 配置和 SSL 材料的目录。有关设置此目录的说明，请参见第 8.4.4.8 节，“使用 keyring_okv KMIP 插件”。

    该目录应具有严格的模式，并且只能由用于运行 MySQL 服务器的帐户访问。例如，在 Unix 和类 Unix 系统上，要使用`/usr/local/mysql/mysql-keyring-okv`目录，以下命令（以`root`身份执行）创建目录并设置其模式和所有权：

    ```sql
    cd /usr/local/mysql
    mkdir mysql-keyring-okv
    chmod 750 mysql-keyring-okv
    chown mysql mysql-keyring-okv
    chgrp mysql mysql-keyring-okv
    ```

    如果分配给`keyring_okv_conf_dir`的值指定了一个不存在的目录，或者不包含启用与 Oracle Key Vault 建立连接的配置信息，`keyring_okv`会将错误消息写入错误日志。如果对`keyring_okv_conf_dir`的尝试运行时赋值导致错误，则变量值和 keyring 操作保持不变。

+   `keyring_operations`

    | 系统变量 | `keyring_operations` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    是否启用了钥匙环操作。此变量在钥匙迁移操作期间使用。参见第 8.4.4.14 节，“在钥匙环密钥存储之间迁移密钥”。修改此变量所需的特权是`ENCRYPTION_KEY_ADMIN`，另外还需要`SYSTEM_VARIABLES_ADMIN`或已弃用的`SUPER`特权。
