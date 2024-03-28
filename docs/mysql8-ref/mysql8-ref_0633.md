> 原文：[`dev.mysql.com/doc/refman/8.0/en/large-page-support.html`](https://dev.mysql.com/doc/refman/8.0/en/large-page-support.html)

#### 10.12.3.3 启用大页支持

一些硬件和操作系统架构支持大于默认值（通常为 4KB）的内存页。此支持的实际实现取决于底层硬件和操作系统。执行大量内存访问的应用程序可能通过使用大页获得性能改进，因为减少了 TLB（Translation Lookaside Buffer）缺失。

在 MySQL 中，大页可以被`InnoDB`使用，为其缓冲池和额外内存池分配内存。

MySQL 中标准使用大页尝试使用支持的最大大小，最高可达 4MB。在 Solaris 下，“超大页”功能使得可以使用高达 256MB 的页面。这个功能适用于最近的 SPARC 平台。可以通过使用`--super-large-pages`或`--skip-super-large-pages`选项来启用或禁用它。

MySQL 还支持 Linux 中的大页支持实现（在 Linux 中称为 HugeTLB）。

在 Linux 上使用大页之前，必须启用内核以支持它们，并且需要配置 HugeTLB 内存池。有关参考，HugeTBL API 在您的 Linux 源代码的`Documentation/vm/hugetlbpage.txt`文件中有文档。

一些最近的系统（如 Red Hat Enterprise Linux）的内核可能默认启用了大页功能。要检查您的内核是否为真，请使用以下命令，并查找包含“huge”的输出行：

```sql
$> grep -i huge /proc/meminfo
AnonHugePages:   2658304 kB
ShmemHugePages:        0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
```

非空的命令输出表示大页支持存在，但零值表示未配置任何页面供使用。

如果您的内核需要重新配置以支持大页，请参考`hugetlbpage.txt`文件中的说明。

假设您的 Linux 内核已启用大页支持，请按以下步骤配置 MySQL 以使用它：

1.  确定所需的大页数。这是 InnoDB 缓冲池大小除以大页大小，我们可以计算为`innodb_buffer_pool_size` / `Hugepagesize`。假设`innodb_buffer_pool_size`的默认值（128MB）并使用从`/proc/meminfo`中获得的`Hugepagesize`值（2MB），这是 128MB / 2MB，或 64 个大页。我们称这个值为*`P`*。

1.  作为系统根用户，在文本编辑器中打开文件`/etc/sysctl.conf`，并添加此处显示的行，其中*`P`*是先前步骤中获得的大页数：

    ```sql
    vm.nr_hugepages=*P*
    ```

    使用先前获得的实际值，附加行应如下所示：

    ```sql
    vm.nr_huge_pages=64
    ```

    保存更新后的文件。

1.  作为系统根用户，运行以下命令：

    ```sql
    $> sudo sysctl -p
    ```

    注意

    在某些系统上，大页文件的名称可能略有不同；例如，某些发行版将其称为`nr_hugepages`。如果**sysctl**返回与文件名相关的错误，请检查`/proc/sys/vm`中相应文件的名称，并使用该名称。

    要验证大页配置，请再次检查`/proc/meminfo`，如前所述。现在，您应该在输出中看到一些额外的非零值，类似于这样：

    ```sql
    $> grep -i huge /proc/meminfo
    AnonHugePages:   2686976 kB
    ShmemHugePages:        0 kB
    HugePages_Total:     233
    HugePages_Free:      233
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    Hugetlb:          477184 kB
    ```

1.  可选地，您可能希望压缩 Linux VM。您可以使用一系列命令来执行此操作，可能是在脚本文件中，类似于以下所示：

    ```sql
    sync
    sync
    sync
    echo 3 > /proc/sys/vm/drop_caches
    echo 1 > /proc/sys/vm/compact_memory
    ```

    查看您的操作平台文档以获取有关如何执行此操作的更多信息。

1.  检查服务器使用的任何配置文件，如`my.cnf`，并确保`innodb_buffer_pool_chunk_size`设置为大于大页大小。此变量的默认值为 128M。

1.  MySQL 服务器中默认情况下禁用大页支持。要启用它，请使用`--large-pages`启动服务器。您还可以通过将以下行添加到服务器`my.cnf`文件的`[mysqld]`部分来执行此操作：

    ```sql
    large-pages=ON
    ```

    启用此选项后，`InnoDB`会自动为其缓冲池和额外内存池使用大页。如果`InnoDB`无法执行此操作，则会回退到使用传统内存，并在错误日志中写入警告：警告：使用传统内存池。

您可以通过在重新启动**mysqld**后再次检查`/proc/meminfo`来验证 MySQL 是否正在使用大页，就像这样：

```sql
$> grep -i huge /proc/meminfo
AnonHugePages:   2516992 kB
ShmemHugePages:        0 kB
HugePages_Total:     233
HugePages_Free:      222
HugePages_Rsvd:       55
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:          477184 kB
```
