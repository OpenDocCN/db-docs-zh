# 10.6.3 优化修复表语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/repair-table-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/repair-table-optimization.html)

`修复表`对于`MyISAM`表类似于使用**myisamchk**进行修复操作，并且一些相同的性能优化也适用：

+   **myisamchk**具有控制内存分配的变量。您可以通过设置这些变量来改善性能，如第 6.6.4.6 节“myisamchk 内存使用”中所述。

+   对于`修复表`，同样的原则适用，但由于修复是由服务器完成的，您需要设置服务器系统变量，而不是**myisamchk**变量。此外，除了设置内存分配变量外，增加`myisam_max_sort_file_size`系统变量的值会增加修复使用更快的文件排序方法并避免较慢的按键缓存方法的可能性。在检查确保有足够的空间来保存表文件的副本后，将变量设置为系统的最大文件大小。空间必须在包含原始表文件的文件系统中可用。

假设使用以下选项设置其内存分配变量进行**myisamchk**表修复操作：

```sql
--key_buffer_size=128M --myisam_sort_buffer_size=256M
--read_buffer_size=64M --write_buffer_size=64M
```

一些**myisamchk**变量对应于服务器系统变量：

| **myisamchk**变量 | 系统变量 |
| --- | --- |
| `key_buffer_size` | `key_buffer_size` |
| `myisam_sort_buffer_size` | `myisam_sort_buffer_size` |
| `read_buffer_size` | `read_buffer_size` |
| `write_buffer_size` | 无 |

每个服务器系统变量都可以在运行时设置，其中一些变量（`myisam_sort_buffer_size`，`read_buffer_size`）除了全局值外还有一个会话值。设置会话值会限制更改的影响范围仅限于当前会话，并不影响其他用户。更改全局变量（`key_buffer_size`，`myisam_max_sort_file_size`）会影响其他用户。对于`key_buffer_size`，您必须考虑到该缓冲区与其他用户共享。例如，如果您将**myisamchk**的`key_buffer_size`变量设置为 128MB，您可以将相应的`key_buffer_size`系统变量设置得比这更大（如果尚未设置更大），以允许其他会话中的活动使用键缓冲区。然而，更改全局键缓冲区大小会使缓冲区无效，导致其他会话的磁盘 I/O 增加并减慢。避免这个问题的替代方法是使用一个单独的键缓存，将要修复的表的索引分配给它，并在修复完成后取消分配。参见 Section 10.10.2.2, “Multiple Key Caches”。

根据前述说明，可以按照以下方式执行`REPAIR TABLE`操作，以使用类似于**myisamchk**命令的设置。在这里，分配了一个单独的 128MB 键缓冲区，并假定文件系统至少允许文件大小为 100GB。

```sql
SET SESSION myisam_sort_buffer_size = 256*1024*1024;
SET SESSION read_buffer_size = 64*1024*1024;
SET GLOBAL myisam_max_sort_file_size = 100*1024*1024*1024;
SET GLOBAL repair_cache.key_buffer_size = 128*1024*1024;
CACHE INDEX *tbl_name* IN repair_cache;
LOAD INDEX INTO CACHE *tbl_name*;
REPAIR TABLE *tbl_name* ;
SET GLOBAL repair_cache.key_buffer_size = 0;
```

如果您打算更改一个全局变量，但只想在`REPAIR TABLE`操作的持续时间内对其他用户的影响最小化，可以将其值保存在用户变量中，并在操作后恢复。例如：

```sql
SET @old_myisam_sort_buffer_size = @@GLOBAL.myisam_max_sort_file_size;
SET GLOBAL myisam_max_sort_file_size = 100*1024*1024*1024;
REPAIR TABLE tbl_name ;
SET GLOBAL myisam_max_sort_file_size = @old_myisam_max_sort_file_size;
```

影响`REPAIR TABLE`的系统变量可以在服务器启动时全局设置，如果您希望这些值默认生效。例如，将以下行添加到服务器的`my.cnf`文件中：

```sql
[mysqld]
myisam_sort_buffer_size=256M
key_buffer_size=1G
myisam_max_sort_file_size=100G
```

这些设置不包括`read_buffer_size`。将`read_buffer_size`全局设置为一个较大的值会影响所有会话，并且可能会因为为具有许多同时会话的服务器进行过多的内存分配而导致性能下降。
