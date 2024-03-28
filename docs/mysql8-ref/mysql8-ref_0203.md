> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisamchk-memory.html`](https://dev.mysql.com/doc/refman/8.0/en/myisamchk-memory.html)

#### 6.6.4.6 myisamchk 内存使用

在运行**myisamchk**时，内存分配非常重要。**myisamchk**使用的内存不会超过其与内存相关的变量设置的值。如果你要在非常大的表上使用**myisamchk**，你应该首先决定要使用多少内存。默认情况下，只使用约 3MB 来执行修复。通过使用更大的值，你可以让**myisamchk**运行得更快。例如，如果你有超过 512MB 的可用 RAM，你可以使用这些选项（除了你可能指定的其他选项）：

```sql
myisamchk --myisam_sort_buffer_size=256M \
           --key_buffer_size=512M \
           --read_buffer_size=64M \
           --write_buffer_size=64M ...
```

使用`--myisam_sort_buffer_size=16M`对大多数情况来说可能已经足够了。

请注意，**myisamchk**在`TMPDIR`中使用临时文件。如果`TMPDIR`指向一个内存文件系统，很容易发生内存不足错误。如果发生这种情况，请使用`--tmpdir=*`dir_name`*`选项运行**myisamchk**来指定一个位于具有更多空间的文件系统上的目录。

在执行修复操作时，**myisamchk**还需要大量的磁盘空间：

+   数据文件的两倍大小（原始文件和副本）。如果你使用`--quick`进行修复，则不需要这个空间；在这种情况下，只重新创建索引文件。*这个空间必须在与原始数据文件相同的文件系统上可用*，因为副本是在与原始文件相同的目录中创建的。

+   用于替换旧索引文件的新索引文件的空间。旧索引文件在修复操作开始时被截断，所以通常忽略这个空间。这个空间必须在与原始数据文件相同的文件系统上可用。

+   当使用`--recover`或`--sort-recover`（但不是使用`--safe-recover`）时，需要磁盘上的排序空间。这个空间在临时目录中分配（由`TMPDIR`或`--tmpdir=*`dir_name`*`指定）。以下公式给出所需空间的量：

    ```sql
    (*largest_key* + *row_pointer_length*) * *number_of_rows* * 2
    ```

    你可以使用**myisamchk -dv *`tbl_name`***来检查键的长度和*`row_pointer_length`*（参见第 6.6.4 节，“使用 myisamchk 获取表信息”）。*`row_pointer_length`*和*`number_of_rows`*的值分别是表描述中的`Datafile pointer`和`Data records`值。要确定*`largest_key`*的值，请检查表描述中的`Key`行。`Len`列指示每个键部分的字节数。对于多列索引，键大小是所有键部分的`Len`值之和。

如果在修复过程中遇到磁盘空间问题，你可以尝试使用`--safe-recover`代替`--recover`。
