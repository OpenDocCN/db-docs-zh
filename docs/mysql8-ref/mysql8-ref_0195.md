# 6.6.2 innochecksum — 离线 InnoDB 文件校验和实用程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innochecksum.html`](https://dev.mysql.com/doc/refman/8.0/en/innochecksum.html)

**innochecksum** 会为`InnoDB`文件打印校验和。该工具读取一个`InnoDB`表空间文件，为每个页面计算校验和，比较计算的校验和与存储的校验和，报告不匹配，这表明有损坏的页面。最初开发此工具是为了加快在断电后验证表空间文件的完整性，但也可用于文件复制后。因为校验和不匹配会导致`InnoDB`有意关闭正在运行的服务器，所以使用此工具可能比等待生产服务器遇到损坏页面更可取。

**innochecksum** 不能用于服务器已经打开的表空间文件。对于这样的文件，应该使用`CHECK TABLE` 来检查表空间中的表。尝试在服务器已经打开的表空间上运行 **innochecksum** 会导致无法锁定文件的错误。

如果发现校验和不匹配，��备份中恢复表空间或启动服务器并尝试使用 **mysqldump** 对表空间中的表进行备份。

运行 **innochecksum** 的方法如下：

```sql
innochecksum [*options*] *file_name*
```

#### innochecksum 选项

**innochecksum** 支持以下选项。对于涉及页码的选项，数字是从零开始的。

+   `--help`, `-?`

    | 命令行格式 | `--help` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示命令行帮助。示例用法：

    ```sql
    innochecksum --help
    ```

+   `--info`, `-I`

    | 命令行格式 | `--info` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    `--help` 的同义词。显示命令行帮助。示例用法：

    ```sql
    innochecksum --info
    ```

+   `--version`, `-V`

    | 命令行格式 | `--version` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    显示版本信息。示例用法：

    ```sql
    innochecksum --version
    ```

+   `--verbose`, `-v`

    | 命令行格式 | `--verbose` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    详细模式；每五秒向日志文件打印进度指示器。为了打印进度指示器，必须使用`--log`选项指定日志文件。要启用`详细`模式，请运行：

    ```sql
    innochecksum --verbose
    ```

    要关闭详细模式，请运行：

    ```sql
    innochecksum --verbose=FALSE
    ```

    `--verbose`选项和`--log`选项可以同时指定。例如：

    ```sql
    innochecksum --verbose --log=/var/lib/mysql/test/logtest.txt
    ```

    要在日志文件中查找进度指示信息，可以执行以下搜索：

    ```sql
    cat ./logtest.txt | grep -i "okay"
    ```

    日志文件中的进度指示信息如下所示：

    ```sql
    page 1663 okay: 2.863% done
    page 8447 okay: 14.537% done
    page 13695 okay: 23.568% done
    page 18815 okay: 32.379% done
    page 23039 okay: 39.648% done
    page 28351 okay: 48.789% done
    page 33023 okay: 56.828% done
    page 37951 okay: 65.308% done
    page 44095 okay: 75.881% done
    page 49407 okay: 85.022% done
    page 54463 okay: 93.722% done
    ...
    ```

+   `--count`, `-c`

    | 命令行格式 | `--count` |
    | --- | --- |
    | 类型 | 基本名称 |
    | 默认值 | `true` |

    打印文件中页面数的计数并退出。示例用法：

    ```sql
    innochecksum --count ../data/test/tab1.ibd
    ```

+   `--start-page=*`num`*`, `-s *`num`*`

    | 命令行格式 | `--start-page=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |

    从此页码开始。示例用法：

    ```sql
    innochecksum --start-page=600 ../data/test/tab1.ibd
    ```

    或：

    ```sql
    innochecksum -s 600 ../data/test/tab1.ibd
    ```

+   `--end-page=*`num`*`, `-e *`num`*`

    | 命令行格式 | `--end-page=#` |
    | --- | --- |
    | 类型 | 数值 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |

    结束于此页码。示例用法：

    ```sql
    innochecksum --end-page=700 ../data/test/tab1.ibd
    ```

    或：

    ```sql
    innochecksum --p 700 ../data/test/tab1.ibd
    ```

+   `--page=*`num`*`, `-p *`num`*`

    | 命令行格式 | `--page=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |

    仅检查此页码。示例用法：

    ```sql
    innochecksum --page=701 ../data/test/tab1.ibd
    ```

+   `--strict-check`, `-C`

    | 命令行格式 | `--strict-check=algorithm` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `crc32` |
    | 有效值 | `innodb``crc32``none` |

    指定严格的校验算法。选项包括`innodb`、`crc32`和`none`。

    在此示例中，指定了`innodb`校验算法：

    ```sql
    innochecksum --strict-check=innodb ../data/test/tab1.ibd
    ```

    在此示例中，指定了`crc32`校验算法：

    ```sql
    innochecksum -C crc32 ../data/test/tab1.ibd
    ```

    以下条件适用：

    +   如果您没有指定`--strict-check`选项，**innochecksum** 将校验`innodb`、`crc32`和`none`。

    +   如果指定`none`选项，则只允许由`none`生成的校验。

    +   如果指定`innodb`选项，则只允许由`innodb`生成的校验。

    +   如果指定`crc32`选项，则只允许由`crc32`生成的校验。

+   `--no-check`, `-n`

    | 命令行格式 | `--no-check` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    重写校验时忽略校验。此选项只能与**innochecksum** 的`--write`选项一起使用。如果未指定`--write`选项，**innochecksum** 将终止。

    在此示例中，将`innodb`校验重写以替换无效校验：

    ```sql
    innochecksum --no-check --write innodb ../data/test/tab1.ibd
    ```

+   `--allow-mismatches`, `-a`

    | 命令行格式 | `--allow-mismatches=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |

    在**innochecksum**终止之前允许的最大校验和不匹配次数。默认设置为 0。如果`--allow-mismatches=`*`N`*，其中`*`N`*>=0，则允许*N*个不匹配，并且**innochecksum**在`*`N`*+1`时终止。当`--allow-mismatches`设置为 0 时，**innochecksum**在第一个校验和不匹配时终止。

    在此示例中，将现有的`innodb`校验和重写为将`--allow-mismatches`设置为 1。

    ```sql
    innochecksum --allow-mismatches=1 --write innodb ../data/test/tab1.ibd
    ```

    当`--allow-mismatches`设置为 1 时，如果在具有 1000 个页面的文件中的第 600 页存在不匹配，然后在第 700 页存在另一个不匹配，校验和将更新为 0-599 页和 601-699 页。因为`--allow-mismatches`设置为 1，校验和容忍第一个不匹配，并在第二个不匹配时终止，使第 600 页和第 700-999 页保持不变。

+   `--write=*`name`*`, `-w *`num`*`

    | 命令行格式 | `--write=algorithm` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `crc32` |
    | 有效值 | `innodb``crc32``none` |

    重写校验和。在重写无效校验和时，必须将`--no-check`选项与`--write`选项一起使用。`--no-check`选项告诉**innochecksum**忽略无效校验和的验证。如果当前校验和有效，则不必指定`--no-check`选项。

    使用`--write`选项时必须指定算法。`--write`选项的可能值包括：

    +   `innodb`: 使用来自`InnoDB`的原始算法在软件中计算的校验和。

    +   `crc32`: 使用`crc32`算法计算的校验和，可能使用硬件辅助完成。

    +   `none`: 一个常数。

    `--write`选项将整个页面重写到磁盘。如果新校验和与现有校验和相同，则为了最小化 I/O，新校验和不会写入磁盘。

    **innochecksum**在使用`--write`选项时获得独占锁。

    在此示例中，为`tab1.ibd`写入`crc32`校验和：

    ```sql
    innochecksum -w crc32 ../data/test/tab1.ibd
    ```

    在此示例中，重写`crc32`校验和以替换无效的`crc32`校验和：

    ```sql
    innochecksum --no-check --write crc32 ../data/test/tab1.ibd
    ```

+   `--page-type-summary`, `-S`

    | 命令行格式 | `--page-type-summary` |
    | --- | --- |
    | 类型 | 布尔值 |
    | 默认值 | `false` |

    在表空间中显示每种页面类型的计数。示例用法：

    ```sql
    innochecksum --page-type-summary ../data/test/tab1.ibd
    ```

    `--page-type-summary`的示例输出：

    ```sql
    File::../data/test/tab1.ibd
    ================PAGE TYPE SUMMARY==============
    #PAGE_COUNT PAGE_TYPE
    ===============================================
           2        Index page
           0        Undo log page
           1        Inode page
           0        Insert buffer free list page
           2        Freshly allocated page
           1        Insert buffer bitmap
           0        System page
           0        Transaction system page
           1        File Space Header
           0        Extent descriptor page
           0        BLOB page
           0        Compressed BLOB page
           0        Other type of page
    ===============================================
    Additional information:
    Undo page type: 0 insert, 0 update, 0 other
    Undo page state: 0 active, 0 cached, 0 to_free, 0 to_purge, 0 prepared, 0 other
    ```

+   `--page-type-dump`, `-D`

    | 命令行格式 | `--page-type-dump=name` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `[无]` |

    将表空间中每个页面的页面类型信息转储到`stderr`或`stdout`。示例用法：

    ```sql
    innochecksum --page-type-dump=/tmp/a.txt ../data/test/tab1.ibd
    ```

+   `--log`, `-l`

    | 命令行格式 | `--log=path` |
    | --- | --- |
    | 类型 | 文件名 |
    | 默认值 | `[无]` |

    innochecksum 工具的日志输出。必须提供日志文件名。日志输出包含每个表空间页面的校验值。对于未压缩的表，还提供了 LSN 值。`--log`替换了早期版本中可用的`--debug`选项。示例用法：

    ```sql
    innochecksum --log=/tmp/log.txt ../data/test/tab1.ibd
    ```

    或者：

    ```sql
    innochecksum -l /tmp/log.txt ../data/test/tab1.ibd
    ```

+   `-` 选项。

    指定`-`选项以从标准输入读取。如果在期望“从标准输入读取”时缺少`-`选项，则 innochecksum 会打印 innochecksum 的使用信息，指示省略了`-`选项。示例用法：

    ```sql
    cat t1.ibd | innochecksum -
    ```

    在这个例子中，innochecksum 将`crc32`校验算法写入`a.ibd`，而不改变原始的`t1.ibd`文件。

    ```sql
    cat t1.ibd | innochecksum --write=crc32 - > a.ibd
    ```

#### 在多个用户定义的表空间文件上运行 innochecksum

以下示例演示如何在多个用户定义的表空间文件（`.ibd`文件）上运行 innochecksum。

运行 innochecksum 来检查“test”数据库中所有表空间（`.ibd`）文件：

```sql
innochecksum ./data/test/*.ibd
```

运行 innochecksum 来检查所有以“t”开头的表空间文件（`.ibd`文件）：

```sql
innochecksum ./data/test/t*.ibd
```

运行 innochecksum 来检查`data`目录中的所有表空间文件（`.ibd`文件）：

```sql
innochecksum ./data/*/*.ibd
```

注意

在 Windows 操作系统上不支持在多个用户定义的表空间文件上运行 innochecksum，因为 Windows shell（如**cmd.exe**）不支持通配符扩展。在 Windows 系统上，必须为每个用户定义的表空间文件单独运行 innochecksum。例如：

```sql
innochecksum.exe t1.ibd
innochecksum.exe t2.ibd
innochecksum.exe t3.ibd
```

#### 在多个系统表空间文件上运行 innochecksum

默认情况下，只有一个 `InnoDB` 系统表空间文件（`ibdata1`），但可以使用 `innodb_data_file_path` 选项定义系统表空间的多个文件。在以下示例中，使用 `innodb_data_file_path` 选项定义了三个系统表空间文件：`ibdata1`、`ibdata2` 和 `ibdata3`。

```sql
./bin/mysqld --no-defaults --innodb-data-file-path="ibdata1:10M;ibdata2:10M;ibdata3:10M:autoextend"
```

三个文件（`ibdata1`、`ibdata2` 和 `ibdata3`）形成一个逻辑系统表空间。要对形成一个逻辑系统表空间的多个文件运行 **innochecksum**，**innochecksum** 需要使用 `-` 选项从标准输入读取表空间文件，这相当于连接多个文件以创建一个单一文件。对于上面提供的示例，将使用以下 **innochecksum** 命令：

```sql
cat ibdata* | innochecksum -
```

有关“-”选项的更多信息，请参考 **innochecksum** 选项信息。

注意

在 Windows 操作系统上不支持在同一表空间中运行 **innochecksum** 多个文件，因为 Windows shells（如 **cmd.exe**）不支持通配符模式扩展。在 Windows 系统上，必须分别对每个系统表空间文件运行 **innochecksum**。例如：

```sql
innochecksum.exe ibdata1
innochecksum.exe ibdata2
innochecksum.exe ibdata3
```
