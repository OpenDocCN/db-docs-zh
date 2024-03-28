# 25.5.31 ndbxfrm — 压缩、解压、加密和解密由 NDB Cluster 创建的文件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbxfrm.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbxfrm.html)

**ndbxfrm** 实用程序，引入于 NDB 8.0.22，可用于解压、解密和输出由 NDB Cluster 创建的文件的信息，这些文件可能被压缩、加密或两者兼有。它还可用于压缩或加密文件。

**表 25.52 与程序 ndbxfrm 一起使用的命令行选项**

| 格式 | 描述 | 新增、弃用或移除 |
| --- | --- | --- |
| `--compress`,`-c` | 压缩文件 | 新增：NDB 8.0.22 |
| `--decrypt-key=key` | 提供文件解密密钥 | 新增：NDB 8.0.31 |
| `--decrypt-key-from-stdin` | 从标准输入中提供文件解密密钥 | 新增：NDB 8.0.31 |
| `--decrypt-password=password` | 使用此密码解密文件 | 新增：NDB 8.0.22 |
| `--decrypt-password-from-stdin` | 从标准输入中以安全方式获取解密密码 | 新增：NDB 8.0.24 |
| `--defaults-extra-file=path` | 在全局文件读取后读取给定文件 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-group-suffix=string` | 也读取带有 concat(group, suffix) 的组 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--defaults-file=path` | 仅从给定文件中读取默认选项 | (在基于 MySQL 8.0 的所有 NDB 版本中支持) |
| `--encrypt-block-size=#` | 打印有关文件的信息，包括文件头和尾部 | 新增：NDB 8.0.31 |
| `--encrypt-block-size=#` | 作为一个单元加密的输入数据块的大小。与 XTS 一起使用，对于 CBC 模式设置为零 | 新增：NDB 8.0.29 |
| `--encrypt-cipher=#` | 加密密码：1 代表 CBC，2 代表 XTS | 新增：NDB 8.0.29 |
| `--encrypt-kdf-iter-count=#`,`-k #` | 密钥定义中使用的迭代次数 | 新增：NDB 8.0.22 |
| `--encrypt-key=key` | 使用此密钥加密文件 | 新增：NDB 8.0.31 |
| `--encrypt-key-from-stdin` | 使用从标准输入提供的密钥加密文件 | 新增：NDB 8.0.31 |
| `--encrypt-password=password` | 使用此密码加密文件 | 新增：NDB 8.0.22 |
| `--encrypt-password-from-stdin` | 从标准输入安全获取加密密码 | 新增：NDB 8.0.24 |
| `--help`,`-?` | 打印使用信息 | 新增：NDB 8.0.22 |
| `--info`,`-i` | 打印文件信息 | 新增：NDB 8.0.22 |
| `--login-path=path` | 从登录文件中读取给定路径 | (支持所有基于 MySQL 8.0 的 NDB 发行版) |
| `--no-defaults` | 不从除登录文件以外的任何选项文件中读取默认选项 | (支持所有基于 MySQL 8.0 的 NDB 发行版) |
| `--print-defaults` | 打印程序参数列表并退出 | (支持所有基于 MySQL 8.0 的 NDB 发行版) |
| `--usage`,`-?` | 打印使用信息；--help 的同义词 | 新增：NDB 8.0.22 |
| `--version`,`-V` | 输出版本信息 | 新增：NDB 8.0.22 |
| 格式 | 描述 | 添加、弃用或移除 |

#### 用法

```sql
ndbxfrm --info *file*[ *file* ...]

ndbxfrm --compress *input_file* *output_file*

ndbxfrm --decrypt-password=*password* *input_file* *output_file*

ndbxfrm [--encrypt-ldf-iter-count=#] --encrypt-password=*password* *input_file* *output_file*
```

*`input_file`* 和 *`output_file`* 不能是同一个文件。

#### 选项

+   `--compress`, `-c`

    | 命令行格式 | `--compress` |
    | --- | --- |
    | 引入版本 | 8.0.22-ndb-8.0.22 |

    压缩输入文件，使用与压缩 NDB 集群备份相同的压缩方法，并将输出写入输出文件。要解压未加密的压缩 `NDB` 备份文件，只需调用 **ndbxfrm**，并提供压缩文件的名称和输出文件的名称（无需任何选项）。

+   `--decrypt-key=*`key`*`, `-K` *`key`*

    | 命令行格式 | `--decrypt-key=key` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    使用提供的密钥解密由`NDB`加密的文件。

    注意

    此选项不能与`--decrypt-password`一起使用。

+   `--decrypt-key-from-stdin`

    | 命令行格式 | `--decrypt-key-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    使用从标准输入提供的密钥解密由`NDB`加密的文件。

+   `--decrypt-password=*`password`*`

    | 命令行格式 | `--decrypt-password=password` |
    | --- | --- |
    | 引入版本 | 8.0.22-ndb-8.0.22 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    使用提供的密码解密由`NDB`加密的文件。

    注意

    此选项不能与`--decrypt-key`一起使用。

+   [`--decrypt-password-from-stdin[=TRUE|FALSE]`](mysql-cluster-programs-ndbxfrm.html#option_ndbxfrm_decrypt-password-from-stdin)

    | 命令行格式 | `--decrypt-password-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.24-ndb-8.0.24 |

    使用从标准输入提供的密码解密由`NDB`加密的文件。这类似于在调用**mysql** `--password`时不跟随选项后输入密码��

+   `--defaults-extra-file`

    | 命令行格式 | `--defaults-extra-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    在全局文件读取后读取给定文件。

+   `--defaults-file`

    | 命令行格式 | `--defaults-file=path` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    仅从给定文件中读取默认选项。

+   `--defaults-group-suffix`

    | 命令行格式 | `--defaults-group-suffix=string` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    还读取带有`CONCAT(*`group`*, *`suffix`*)`的组。

+   `--detailed-info`

    | 命令行格式 | `--encrypt-block-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |
    | 类型 | 布尔值 |
    | 默认值 | `FALSE` |

    打印出文件信息，类似于`--info`，但包括文件的头部和尾部。

    示例：

    ```sql
    $> ndbxfrm --detailed-info S0.sysfile
    File=/var/lib/cluster-data/ndb_7_fs/D1/NDBCNTR/S0.sysfile, compression=no, encryption=yes
    header: {
      fixed_header: {
        magic: {
          magic: { 78, 68, 66, 88, 70, 82, 77, 49 },
          endian: 18364758544493064720,
          header_size: 32768,
          fixed_header_size: 160,
          zeros: { 0, 0 }
        },
        flags: 73728,
        flag_extended: 0,
        flag_zeros: 0,
        flag_file_checksum: 0,
        flag_data_checksum: 0,
        flag_compress: 0,
        flag_compress_method: 0,
        flag_compress_padding: 0,
        flag_encrypt: 18,
        flag_encrypt_cipher: 2,
        flag_encrypt_krm: 1,
        flag_encrypt_padding: 0,
        flag_encrypt_key_selection_mode: 0,
        dbg_writer_ndb_version: 524320,
        octets_size: 32,
        file_block_size: 32768,
        trailer_max_size: 80,
        file_checksum: { 0, 0, 0, 0 },
        data_checksum: { 0, 0, 0, 0 },
        zeros01: { 0 },
        compress_dbg_writer_header_version: { ... },
        compress_dbg_writer_library_version: { ... },
        encrypt_dbg_writer_header_version: { ... },
        encrypt_dbg_writer_library_version: { ... },
        encrypt_key_definition_iterator_count: 100000,
        encrypt_krm_keying_material_size: 32,
        encrypt_krm_keying_material_count: 1,
        encrypt_key_data_unit_size: 32768,
        encrypt_krm_keying_material_position_in_octets: 0,
      },
      octets: {
         102, 68, 56, 125, 78, 217, 110, 94, 145, 121, 203, 234, 26, 164, 137, 180,
         100, 224, 7, 88, 173, 123, 209, 110, 185, 227, 85, 174, 109, 123, 96, 156,
      }
    }
    trailer: {
      fixed_trailer: {
        flags: 48,
        flag_extended: 0,
        flag_zeros: 0,
        flag_file_checksum: 0,
        flag_data_checksum: 3,
        data_size: 512,
        file_checksum: { 0, 0, 0, 0 },
        data_checksum: { 226, 223, 102, 207 },
        magic: {
          zeros: { 0, 0 }
          fixed_trailer_size: 56,
          trailer_size: 32256,
          endian: 18364758544493064720,
          magic: { 78, 68, 66, 88, 70, 82, 77, 49 },
        },
      }
    }
    ```

+   `--encrypt-block-size=*`#`*`

    | 命令行格式 | `--encrypt-block-size=#` |
    | --- | --- |
    | 引入版本 | 8.0.29-ndb-8.0.29 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    作为一个单元加密的输入数据块的大小。与 XTS 一起使用；对于 CBC 模式，设置为`0`（默认值）。

+   `--encrypt-cipher=*`#`*`

    | 命令行格式 | `--encrypt-cipher=#` |
    | --- | --- |
    | 引入版本 | 8.0.29-ndb-8.0.29 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    用于加密的密码。设置为`1`表示 CBC 模式（默认），或`2`表示 XTS。

+   `--encrypt-kdf-iter-count=*`#`*`, `-k *`#`*`

    | 命令行格式 | `--encrypt-kdf-iter-count=#` |
    | --- | --- |
    | 引入版本 | 8.0.22-ndb-8.0.22 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `2147483647` |

    在加密文件时，指定用于加密密钥的迭代次数。需要`--encrypt-password`选项。

+   `--encrypt-key=*`key`*`

    | 命令行格式 | `--encrypt-key=key` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    使用提供的密钥加密文件。

    注意

    此选项不能与`--encrypt-password`一起使用。

+   `--encrypt-key-from-stdin`

    | 命令行格式 | `--encrypt-key-from-stdin` |
    | --- | --- |
    | 引入版本 | 8.0.31-ndb-8.0.31 |

    使用从`stdin`提供的密钥加密文件。

+   `--encrypt-password=*`password`*`

    | 命令行格式 | `--encrypt-password=password` |
    | --- | --- |
    | 引入版本 | 8.0.22-ndb-8.0.22 |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    使用选项提供的密码加密备份文件。密码必须符合此处列出的要求：

    +   使用除`!`、`'`、`"`、`$`、`%`、`\`之外的任何可打印 ASCII 字符

    $> ndbxfrm -i BACKUP-10-0.5.Data BACKUP-10.5.ctl BACKUP-10.5.log

    文件=BACKUP-10-0.5.Data，压缩=no，加密=yes

    文件=BACKUP-10.5.ctl，压缩=no，加密=yes

    文件=BACKUP-10.5.log，压缩=no，加密=yes

    ```sql

    Beginning with NDB 8.0.31, you can also see the file's header and trailer using the `--detailed-info` option.

*   `--login-path`

    | Command-Line Format | `--login-path=path` |
    | Type | String |
    | Default Value | `[none]` |

    Read given path from login file.

*   `--no-defaults`

    | Command-Line Format | `--no-defaults` |

    Do not read default options from any option file other than login file.

*   `--print-defaults`

    | Command-Line Format | `--print-defaults` |

    Print program argument list and exit.

*   `--usage`, `-?`

    | Command-Line Format | `--usage` |
    | Introduced | 8.0.22-ndb-8.0.22 |

    Synonym for `--help`.

*   `--version`, `-V`

    | Command-Line Format | `--version` |
    | Introduced | 8.0.22-ndb-8.0.22 |

    Prints out version information.

**ndbxfrm** can encrypt backups created by any version of NDB Cluster. The `.Data`, `.ctl`, and `.log` files comprising the backup must be encrypted separately, and these files must be encrypted separately for each data node. Once encrypted, such backups can be decrypted only by **ndbxfrm**, **ndb_restore**, or **ndb_print_backup** from NDB Cluster 8.0.22 or later.

An encrypted file can be re-encrypted with a new password using the `--encrypt-password` and `--decrypt-password` options together, like this:

```

ndbxfrm --decrypt-password=*old* --encrypt-password=*new* *input_file* *output_file*

```

在刚才显示的示例中，*`old`*和*`new`*分别是旧密码和新密码；这两者都必须用引号括起来。输入文件被解密，然后作为输出文件被加密。输入文件本身不会改变；如果您不希望使用旧密码访问它，必须手动删除输入文件。
