# 6.6.1 ibd2sdi — InnoDB 表空间 SDI 提取实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/ibd2sdi.html`](https://dev.mysql.com/doc/refman/8.0/en/ibd2sdi.html)

**ibd2sdi**是一个用于从`InnoDB`表空间文件中提取序列化字典信息（SDI）的实用程序。SDI 数据存在于所有持久的`InnoDB`表空间文件中。

**ibd2sdi**可以运行在 file-per-table 表空间文件（`*.ibd`文件）、general tablespace 文件（`*.ibd`文件）、system tablespace 文件（`ibdata*`文件）和数据字典表空间（`mysql.ibd`）上。不支持临时表空间或撤销表空间的使用。

**ibd2sdi**可以在运行时或服务器离线时使用。在与 SDI 相关的 DDL 操作、`ROLLBACK`操作和撤销日志清除操作期间，可能会有一个短暂的时间间隔，此时**ibd2sdi**无法读取存储在表空间中的 SDI 数据。

**ibd2sdi**从指定的表空间执行未提交的 SDI 读取。不访问重做日志和撤销日志。

调用**ibd2sdi**实用程序如下：

```sql
ibd2sdi [*options*] *file_name1* [*file_name2 file_name3 ...*]
```

**ibd2sdi**支持像`InnoDB`系统表空间这样的多文件表空间，但不能同时运行多个表空间。对于多文件表空间，请指定每个文件：

```sql
ibd2sdi ibdata1 ibdata2
```

多文件表空间的文件必须按照升序页号的顺序指定。如果两个连续的文件具有相同的空间 ID，则后一个文件必须以前一个文件的最后页号+1 开始。

**ibd2sdi**以`JSON`格式输出包含 id、类型和数据字段的 SDI。

#### ibd2sdi 选项

**ibd2sdi**支持以下选项：

+   `--help`, `-h`

    | 命令行格式 | `--help` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示帮助消息并退出。例如：

    ```sql
    Usage: ./ibd2sdi [-v] [-c <strict-check>] [-d <dump file name>] [-n] filename1 [filenames]
    See http://dev.mysql.com/doc/refman/8.0/en/ibd2sdi.html for usage hints.
      -h, --help          Display this help and exit.
      -v, --version       Display version information and exit.
      -#, --debug[=name]  Output debug log. See
                          http://dev.mysql.com/doc/refman/8.0/en/dbug-package.html
      -d, --dump-file=name
                          Dump the tablespace SDI into the file passed by user.
                          Without the filename, it will default to stdout
      -s, --skip-data     Skip retrieving data from SDI records. Retrieve only id
                          and type.
      -i, --id=#          Retrieve the SDI record matching the id passed by user.
      -t, --type=#        Retrieve the SDI records matching the type passed by
                          user.
      -c, --strict-check=name
                          Specify the strict checksum algorithm by the user.
                          Allowed values are innodb, crc32, none.
      -n, --no-check      Ignore the checksum verification.
      -p, --pretty        Pretty format the SDI output.If false, SDI would be not
                          human readable but it will be of less size
                          (Defaults to on; use --skip-pretty to disable.)

    Variables (--variable-name=value)
    and boolean options {FALSE|TRUE}  Value (after reading options)
    --------------------------------- ----------------------------------------
    debug                             (No default value)
    dump-file                         (No default value)
    skip-data                         FALSE
    id                                0
    type                              0
    strict-check                      crc32
    no-check                          FALSE
    pretty                            TRUE
    ```

+   `--version`, `-v`

    | 命令行格式 | `--version` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示版本信息并退出。例如：

    ```sql
    ibd2sdi  Ver 8.0.3-dmr for Linux on x86_64 (Source distribution)
    ```

+   [`--debug[=*`debug_options`*]`](ibd2sdi.html#option_ibd2sdi_debug), `-# [*`debug_options`*]`

    | 命令行格式 | `--debug=options` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[none]` |

    打印调试日志。有关调试选项，请参考第 7.9.4 节，“DBUG 包”。

    ```sql
    ibd2sdi --debug=d:t /tmp/ibd2sdi.trace
    ```

    仅当 MySQL 使用`WITH_DEBUG`构建时才可用此选项。由 Oracle 提供的 MySQL 发布二进制文件*不*使用此选项构建。

+   `--dump-file=`, `-d`

    | 命令行格式 | `--dump-file=file` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[none]` |

    将序列化的字典信息（SDI）转储到指定的转储文件中。如果未指定转储文件，则表空间 SDI 将转储到`stdout`。

    ```sql
    ibd2sdi --dump-file=*file_name* ../data/test/t1.ibd
    ```

+   `--skip-data`, `-s`

    | 命令行格式 | `--skip-data` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    跳过从序列化字典信息（SDI）中检索`data`字段值，仅检索`id`和`type`字段值，这些字段值是 SDI 记录的主键。

    ```sql
    $> ibd2sdi --skip-data ../data/test/t1.ibd
    ["ibd2sdi"
    ,
    {
    	"type": 1,
    	"id": 330
    }
    ,
    {
    	"type": 2,
    	"id": 7
    }
    ]
    ```

+   `--id=*`#`*`, `-i *`#`*`

    | 命令行格式 | `--id=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |

    检索与指定表或表空间对象 ID 匹配的序列化字典信息（SDI）。对象 ID 对于对象类型是唯一的。表和表空间对象 ID 也可以在`mysql.tables`和`mysql.tablespace`数据字典表的`id`列中找到。有关数据字典表的信息，请参见第 16.1 节，“数据字典模式”。

    ```sql
    $> ibd2sdi --id=7 ../data/test/t1.ibd
    ["ibd2sdi"
    ,
    {
    	"type": 2,
    	"id": 7,
    	"object":
    		{
        "mysqld_version_id": 80003,
        "dd_version": 80003,
        "sdi_version": 1,
        "dd_object_type": "Tablespace",
        "dd_object": {
            "name": "test/t1",
            "comment": "",
            "options": "",
            "se_private_data": "flags=16417;id=2;server_version=80003;space_version=1;",
            "engine": "InnoDB",
            "files": [
                {
                    "ordinal_position": 1,
                    "filename": "./test/t1.ibd",
                    "se_private_data": "id=2;"
                }
            ]
        }
    }
    }
    ]
    ```

+   `--type=*`#`*`, `-t *`#`*`

    | 命令行格式 | `--type=#` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `0` |
    | 有效值 | `1``2` |

    检索与指定对象类型匹配的序列化字典信息（SDI）。SDI 提供了表（类型=1）和表空间（类型=2）对象的信息。

    此示例显示了`test`数据库中表空间`ts1`的输出：

    ```sql
    $> ibd2sdi --type=2 ../data/test/ts1.ibd
    ["ibd2sdi"
    ,
    {
    	"type": 2,
    	"id": 7,
    	"object":
    		{
        "mysqld_version_id": 80003,
        "dd_version": 80003,
        "sdi_version": 1,
        "dd_object_type": "Tablespace",
        "dd_object": {
            "name": "test/ts1",
            "comment": "",
            "options": "",
            "se_private_data": "flags=16417;id=2;server_version=80003;space_version=1;",
            "engine": "InnoDB",
            "files": [
                {
                    "ordinal_position": 1,
                    "filename": "./test/ts1.ibd",
                    "se_private_data": "id=2;"
                }
            ]
        }
    }
    }
    ]
    ```

    由于`InnoDB`处理默认值元数据的方式，即使在给定表列的**ibd2sdi**输出中，也可能存在并且非空的默认值，即使未使用`DEFAULT`定义。考虑使用以下语句在名为`i`的数据库中创建的两个表：

    ```sql
    CREATE TABLE t1 (c VARCHAR(16) NOT NULL);

    CREATE TABLE t2 (c VARCHAR(16) NOT NULL DEFAULT "Sakila");
    ```

    使用**ibd2sdi**，我们可以看到列`c`的`default_value`是非空的，并且实际上在两个表中都填充到了指定长度，如下所示：

    ```sql
    $> ibd2sdi ../data/i/t1.ibd  | grep -m1 '\"default_value\"' | cut -b34- | sed -e s/,//
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\nAAAAAAAAAAA="

    $> ibd2sdi ../data/i/t2.ibd  | grep -m1 '\"default_value\"' | cut -b34- | sed -e s/,//
    "BlNha2lsYQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\nAAAAAAAAAAA="
    ```

    使用类似**[jq](https://stedolan.github.io/jq/)**这样的 JSON 感知工具，可以更轻松地检查 **ibd2sdi** 的输出，如下所示：

    ```sql
    $> ibd2sdi ../data/i/t1.ibd  | jq '.[1]["object"]["dd_object"]["columns"][0]["default_value"]'
    "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\nAAAAAAAAAAA="

    $> ibd2sdi ../data/i/t2.ibd  | jq '.[1]["object"]["dd_object"]["columns"][0]["default_value"]'
    "BlNha2lsYQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\nAAAAAAAAAAA="
    ```

    欲了解更多信息，请参阅 MySQL 内部文档。

+   `--strict-check`, `-c`

    | 命令行格式 | `--strict-check=algorithm` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `crc32` |
    | 有效值 | `crc32``innodb``none` |

    指定一个严格的校验算法，用于验证读取页面的校验和。选项包括`innodb`、`crc32`和`none`。

    在这个例子中，指定了`innodb`校验算法的严格版本：

    ```sql
    ibd2sdi --strict-check=innodb ../data/test/t1.ibd
    ```

    在这个例子中，指定了`crc32`校验算法的严格版本：

    ```sql
    ibd2sdi -c crc32 ../data/test/t1.ibd
    ```

    如果不指定`--strict-check`选项，则会根据非严格的`innodb`、`crc32`和`none`校验进行验证。

+   `--no-check`, `-n`

    | 命令行格式 | `--no-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    跳过读取页面时的校验和验证。

    ```sql
    ibd2sdi --no-check ../data/test/t1.ibd
    ```

+   `--pretty`, `-p`

    | 命令行格式 | `--pretty` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    以 JSON 格式输出 SDI 数据。默认启用。如果禁用，则 SDI 不易阅读但文件大小更小。使用`--skip-pretty`来禁用。

    ```sql
    ibd2sdi --skip-pretty ../data/test/t1.ibd
    ```
