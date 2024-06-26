# 概述

> 原文：[`sqlite.com/malloc.html`](https://sqlite.com/malloc.html)

SQLite 使用动态内存分配来获取存储各种对象（例如：数据库连接和预编译语句）的内存，并构建数据库文件的内存缓存以及保存查询结果。为使 SQLite 的动态内存分配子系统可靠、可预测、健壮、安全和高效，做了大量努力。

本文概述了 SQLite 中的动态内存分配。目标受众是在需求高的环境中调整 SQLite 使用以实现最佳性能的软件工程师。使用 SQLite 不需要对本文档中的任何内容有所了解。SQLite 的默认设置和配置通常在大多数应用程序中表现良好。但是，本文档中的信息可能对调整 SQLite 以满足特殊要求或在不寻常情况下运行的工程师有所帮助。

# 1\. 特性

SQLite 核心及其内存分配子系统提供以下功能：

+   **抗分配失败的能力。** 如果内存分配失败（即 malloc()或 realloc()返回 NULL），SQLite 将会优雅地恢复。SQLite 首先尝试释放未固定的缓存页上的内存，然后重试分配请求。如果仍然失败，SQLite 将停止当前操作并将 SQLITE_NOMEM 错误代码传递给应用程序，或者在无法获取请求的内存时尝试其他方式。

+   **无内存泄漏。** 应用程序负责销毁其分配的任何对象。（例如，应用程序必须对每个准备语句使用 sqlite3_finalize()，对每个数据库连接使用 sqlite3_close()。）但只要应用程序配合，SQLite 就不会泄漏内存。即使在内存分配失败或其他系统错误的情况下也是如此。

+   **内存使用限制。** 应用程序可以使用 sqlite3_soft_heap_limit64()机制设置 SQLite 努力保持在其下的内存使用限制。当 SQLite 接近软限制时，它将尝试重用其缓存中的内存，而不是分配新的内存。

+   **零分配选项。** 应用程序可以选择在启动时向 SQLite 提供几个批量内存缓冲区，然后 SQLite 将使用这些提供的缓冲区来满足其所有内存分配需求，从而不调用系统的 malloc()或 free()。

+   **应用程序提供的内存分配器。** 应用程序可以在启动时向 SQLite 提供替代内存分配器的指针。替代内存分配器将用于代替系统的 malloc()和 free()。

+   **防止故障和碎片化。** SQLite 可以配置为在满足以下特定使用约束条件时，保证不会因内存分配错误或堆碎片化而导致系统失败。这一特性对于长时间运行、高可靠性的嵌入式系统非常重要。

+   **内存使用统计信息。** 应用程序可以查看自己使用了多少内存，并检测内存使用是否接近或超出设计边界。

+   **与内存调试工具兼容。** SQLite 中的内存分配结构设计得可以使用标准的第三方内存调试工具（如[dmalloc](http://dmalloc.com)或[valgrind](http://valgrind.org)）来验证正确的内存分配行为。

+   **分配器的最小调用。** 许多系统上的系统 malloc()和 free()实现效率低下。SQLite 通过尽量减少对 malloc()和 free()的使用来减少总体处理时间。

+   **开放访问。** 可插入的 SQLite 扩展甚至应用程序本身可以通过 sqlite3_malloc()、sqlite3_realloc()和 sqlite3_free()接口访问与 SQLite 使用相同的底层内存分配例程。

# 2\. 测试

SQLite 源代码树中的大部分代码都专注于测试和验证。可靠性对 SQLite 很重要。测试基础设施的任务之一是确保 SQLite 不会误用动态分配的内存，不会泄露内存，并且在动态内存分配失败时，SQLite 能够正确响应。

测试基础设施验证 SQLite 在使用特别的内存分配器时不会误用动态分配的内存。在编译时，通过 SQLITE_MEMDEBUG 选项启用了这个特别的工具化内存分配器。这个工具化内存分配器比默认的内存分配器慢得多，因此不建议在生产环境中使用。但在测试期间启用时，工具化内存分配器执行以下检查：

+   **边界检查。** 工具化内存分配器在每个内存分配的两端放置哨兵值，以验证 SQLite 内部是否在分配的边界之外写入数据。

+   **释放后继续使用内存。** 当每个内存块被释放时，每个字节都被覆写为一个无意义的位模式。这有助于确保释放后永远不会再使用内存。

+   **释放非 malloc 获取的内存。** 仪器化内存分配器中的每个内存分配包含用于验证每个释放的分配是否来自之前的 malloc 的标志。

+   **未初始化的内存。** 仪器化内存分配器将每个内存分配初始化为一个无意义的位模式，以帮助确保用户不对分配内存的内容做出任何假设。

无论是否使用仪器化的内存分配器，SQLite 都会跟踪当前分配了多少内存。用于测试 SQLite 的测试脚本有数百个。在每个脚本结束时，所有对象都被销毁，并进行测试以确保所有内存都已释放。这就是如何检测内存泄漏的。请注意，内存泄漏检测始终生效，无论是在测试构建期间还是在生产构建期间。每当开发人员运行任何单个测试脚本时，内存泄漏检测都会激活。因此，在开发过程中出现的内存泄漏会被快速检测和修复。

SQLite 对内存耗尽（OOM）错误的响应进行了测试，使用了一种特殊的内存分配器叠加层来模拟内存失败。该叠加层位于内存分配器和 SQLite 其余部分之间，大部分内存分配请求直接通过叠加层传递给底层分配器，并将结果返回给请求者。但是，叠加层可以被设置为在第 N 次内存分配时失败。进行 OOM 测试时，首先将叠加层设置为在第一次分配尝试时失败。然后运行某些测试脚本，并验证是否正确捕获和处理了分配。然后将叠加层设置为在第二次分配时失败，测试重复进行。失败点会逐次前进，直到整个测试过程完成且未出现内存分配错误。整个测试序列运行两次。第一次运行时，叠加层仅设置为在第 N 次分配时失败。第二次运行时，叠加层设置为在第 N 次及其后续分配时失败。

需要注意的是，即使在使用 OOM 叠加层时，内存泄漏检测逻辑仍然有效。这验证了在遇到内存分配错误时，SQLite 不会泄漏内存。还要注意，OOM 叠加层可以与任何底层内存分配器一起使用，包括检查内存分配误用的仪器化内存分配器。通过这种方式验证了 OOM 错误不会引起其他类型的内存使用错误。

最后，我们观察到，仪器化的内存分配器和内存泄漏检测器在整个 SQLite 测试套件和 TCL 测试套件上均有效，提供超过 99%的语句测试覆盖率，并且 TH3 测试工具提供了 100%的分支测试覆盖率且无泄漏。这充分证明了在 SQLite 内部，动态内存分配被正确使用。

## 2.1\. 使用 reallocarray()

reallocarray() 接口是 OpenBSD 社区的一个新创新（约于 2014 年），起源于努力避免在内存分配大小计算中遇到 32 位整数算术溢出的下一个 ["heartbleed" bug](http://heartbleed.com)。reallocarray() 函数具有单元大小和计数参数。为了分配足够容纳 N 个每个大小为 X 字节的元素的内存数组，可以调用 "reallocarray(0,X,N)"。这比传统的调用 "malloc(X*N)" 更可取，因为 reallocarray() 可以消除 X*N 乘法溢出的风险，并导致 malloc() 返回一个与应用程序预期的大小不同的缓冲区。

SQLite 不使用 reallocarray()。原因是 reallocarray() 对 SQLite 没有用处。事实上，SQLite 从不进行简单的两个整数乘积的内存分配。相反，SQLite 进行的是形如 "X+C"、"N*X+C"、"M*N*X+C" 或 "N*X+M*Y+C" 等形式的分配，reallocarray() 接口在这些情况下无法帮助避免整数溢出问题。

尽管如此，SQLite 在计算内存分配大小时的整数溢出仍然是一个值得关注的问题。为了预防问题，SQLite 所有的内部内存分配都使用了带有 64 位有符号整数大小参数的薄包装函数。SQLite 源代码经过审核，确保所有大小计算都使用 64 位有符号整数进行。SQLite 将拒绝一次性分配超过约 2GB 的内存。（在常规使用中，SQLite 很少一次性分配超过约 8KB 的内存，因此 2GB 的分配限制并不是一个负担。）因此，64 位大小参数为检测溢出提供了充足的余量。进行相同审核的同时，还验证了在计算过程中不可能溢出 64 位整数。

在每次 SQLite 发布之前，都会进行代码审计，以确保 SQLite 中的内存分配大小计算不会溢出。

# 3\. 配置

SQLite 中的默认内存分配设置适用于大多数应用程序。但是，具有非常规或特别严格要求的应用程序可能希望调整配置，以更接近其需求。提供了编译时和启动时的配置选项。

## 3.1\. 替代低级别内存分配器

SQLite 源代码包含几个不同的内存分配模块，可以在编译时选择，或在有限的情况下在启动时选择。

### 3.1.1\. 默认内存分配器

默认情况下，SQLite 使用标准 C 库中的 malloc()、realloc()和 free()例程进行内存分配。这些例程被一个薄包装器包围，还提供了一个"memsize()"函数，该函数返回现有分配的大小。memsize()函数用于准确计算未释放内存的字节数；当释放分配时，memsize()确定从未释放计数中删除多少字节。默认分配器通过在每次 malloc()请求时始终分配额外的 8 个字节，并将分配大小存储在该 8 字节的头部来实现 memsize()。

对于大多数应用程序，建议使用默认内存分配器。如果没有强烈的需求使用替代内存分配器，则使用默认分配器。

### 3.1.2\. 调试内存分配器

如果 SQLite 在编译时使用了 SQLITE_MEMDEBUG 选项，则会使用不同的重型包装器来包装系统的 malloc()、realloc()和 free()函数。这个重型包装器在每次分配时会额外分配大约 100 字节的空间。这些额外的空间用于在返回给 SQLite 核心的分配内存的两端放置哨兵值。当一个分配被释放时，会检查这些哨兵值，以确保 SQLite 核心没有在任何方向上溢出缓冲区。当系统库是 GLIBC 时，这个重型包装器还会利用 GNU backtrace()函数来检查堆栈并记录 malloc()调用的祖先函数。在运行 SQLite 测试套件时，这个重型包装器还会记录当前测试用例的名称。这后两个功能对于跟踪测试套件检测到的内存泄漏的来源非常有用。

当设置 SQLITE_MEMDEBUG 时使用的重型包装器还确保每个新分配的内存在返回给调用者之前都填充了无意义的数据。一旦一个分配被释放，它再次被填充为无意义的数据。这两个动作有助于确保 SQLite 核心不对新分配内存的状态做出假设，并且释放后的内存分配不会再被使用。

SQLITE_MEMDEBUG 使用的重型包装器仅用于 SQLite 的测试、分析和调试。重型包装器具有显著的性能和内存开销，可能不应在生产环境中使用。

### 3.1.3\. Win32 本地内存分配器

如果 SQLite 在 Windows 上使用 SQLITE_WIN32_MALLOC 编译时选项，则会使用不同的薄包装器封装 HeapAlloc()、HeapReAlloc() 和 HeapFree()。这个薄包装器使用配置的 SQLite 堆，如果使用了 SQLITE_WIN32_HEAP_CREATE 编译时选项，则会与默认进程堆不同。此外，如果 SQLite 编译时启用了 assert() 和 SQLITE_WIN32_MALLOC_VALIDATE 编译时选项，将在分配或释放时调用 HeapValidate()。

### 3.1.4\. 零分配内存分配器

当 SQLite 使用 SQLITE_ENABLE_MEMSYS5 选项编译时，会包含一种不使用 malloc() 的替代内存分配器。SQLite 开发者称这种替代内存分配器为 "memsys5"。即使在构建时包含了它，memsys5 默认是禁用的。要启用 memsys5，应用程序必须在启动时调用以下 SQLite 接口：

> ```sql
> sqlite3_config(SQLITE_CONFIG_HEAP, pBuf, szBuf, mnReq);
> 
> ```

在上述调用中，pBuf 是指向一个大的连续内存块的指针，SQLite 将使用它来满足所有的内存分配需求。pBuf 可能指向静态数组，也可能是从其他应用程序特定机制获取的内存。szBuf 是一个整数，表示 pBuf 指向的内存空间的字节数。mnReq 是另一个整数，表示分配的最小大小。任何对 sqlite3_malloc(N) 的调用，其中 N 小于 mnReq，都会向上舍入到 mnReq。mnReq 必须是二的幂。我们将在稍后看到，mnReq 参数在减少 **n** 的值以及在 Robson 证明 中最小内存大小要求方面非常重要。

memsys5 分配器被设计用于嵌入式系统，虽然没有阻止在工作站上使用。szBuf 的大小通常在几百千字节到几十兆字节之间，取决于系统要求和内存预算。

memsys5 使用的算法可以被称为“二的幂，首次适配”。所有内存分配请求的大小都被舍入为二的幂，并且请求由 pBuf 中第一个足够大的空闲插槽满足。使用伙伴系统对相邻的已释放分配进行合并。当适当使用时，该算法提供了对碎片和故障的数学保证，如下面进一步描述的链接。

### 3.1.5\. 实验性内存分配器

为零分配内存分配器使用的名称“memsys5”意味着还有其他几个可用的内存分配器，实际上确实如此。默认内存分配器是“memsys1”。调试内存分配器是“memsys2”。这些已经涵盖过了。

如果 SQLite 编译时启用了 SQLITE_ENABLE_MEMSYS3，那么另一个类似于 memsys5 的零 malloc 内存分配器将包含在源码树中。与 memsys5 类似，memsys3 分配器必须通过调用 sqlite3_config(SQLITE_CONFIG_HEAP,...)来激活。Memsys3 使用提供的内存缓冲区作为所有内存分配的源。Memsys3 与 memsys5 的不同之处在于，memsys3 使用了一种在实践中表现良好的不同内存分配算法，但不提供针对内存碎片化和崩溃的数学保证。Memsys3 是 memsys5 的前身。SQLite 开发人员现在认为 memsys5 优于 memsys3，并且所有需要零 malloc 内存分配器的应用程序应优先使用 memsys5。Memsys3 被视为实验性的和已弃用的，在将来的 SQLite 版本中可能会从源码树中删除。

Memsys4 和 memsys6 是在大约 2007 年引入的实验性内存分配器，随后在大约 2008 年从源码树中移除，因为明显没有增加新的价值。

其他实验性的内存分配器可能会在将来的 SQLite 版本中添加。可以预期这些分配器将被称为 memsys7、memsys8 等。

### 3.1.6\. 应用程序定义的内存分配器

新的内存分配器不必成为 SQLite 源码树的一部分，也不必包含在 sqlite3.c 的合并中。个别应用程序可以在启动时向 SQLite 提供自己的内存分配器。

要使 SQLite 使用新的内存分配器，应用程序只需调用：

> ```sql
> sqlite3_config(SQLITE_CONFIG_MALLOC, pMem);
> 
> ```

在上述调用中，pMem 是指向定义了特定于应用程序的内存分配器接口的 sqlite3_mem_methods 对象的指针。sqlite3_mem_methods 对象实际上只是一个包含指向实现各种内存分配基元函数的函数指针的结构体。

在多线程应用程序中，只有在启用 SQLITE_CONFIG_MEMSTATUS 时，对 sqlite3_mem_methods 的访问才会串行化。如果禁用 SQLITE_CONFIG_MEMSTATUS，那么 sqlite3_mem_methods 中的方法必须自行处理其串行化需求。

### 3.1.7\. 内存分配器叠加

应用程序可以在 SQLite 核心与底层内存分配器之间插入层或“覆盖物”。例如，SQLite 的内存耗尽测试逻辑使用的覆盖物可以模拟内存分配失败。

可以通过使用

> ```sql
> sqlite3_config(SQLITE_CONFIG_GETMALLOC, pOldMem);
> 
> ```

用于获取指向现有内存分配器的指针的接口。覆盖物会保存现有的分配器，并作为执行真实内存分配的备用方式。然后，使用 sqlite3_config(SQLITE_CONFIG_MALLOC,...)将覆盖物插入到现有内存分配器的位置，如上文所述的方式(上文).

### 3.1.8\. 无操作内存分配器存根

如果 SQLite 被编译时选择了 SQLITE_ZERO_MALLOC 选项，那么默认内存分配器将被省略，并被一个只返回“没有可用内存”的存根内存分配器所替代。对存根内存分配器的任何调用都将报告没有可用内存。

无操作内存分配器本身并不实用。它只存在作为一个占位符，以便 SQLite 在可能没有标准库中的 malloc(), free(), or realloc()的系统上链接的内存分配器。使用 SQLITE_ZERO_MALLOC 编译的应用程序将需要使用 sqlite3_config()以及 SQLITE_CONFIG_MALLOC 或 SQLITE_CONFIG_HEAP 来指定在开始使用 SQLite 之前要使用的新替代内存分配器。

## 3.2\. 页面缓存内存

在大多数应用程序中，SQLite 中的数据库页面缓存子系统使用的动态分配内存比 SQLite 的所有其他部分加起来的内存还要多。看到数据库页面缓存消耗的内存超过 SQLite 其余部分的 10 倍以上并不罕见。

可以配置 SQLite 使页面缓存内存从一个独立的和不同的固定大小插槽的内存池中进行分配。这样做有两个优点：

+   由于所有分配的大小都是相同的，内存分配器可以运行得更快。分配器不需要烦恼于合并相邻的空闲插槽或搜索合适大小的插槽。所有未分配的内存插槽可以存储在一个链表上。分配包括从列表中移除第一个条目。释放只是将一个条目添加到列表的开头。

+   使用单一分配大小，Robson proof 中的**n**参数为 1，分配器所需的总内存空间(**N**)正好等于最大内存使用(**M**)。不需要额外的内存来覆盖碎片化开销，从而减少内存需求。对于页面缓存内存来说，这一点尤为重要，因为页面缓存构成了 SQLite 内存需求中最大的组件。

页面缓存内存分配器默认情况下是禁用的。应用程序可以在启动时启用它，如下所示：

> ```sql
> sqlite3_config(SQLITE_CONFIG_PAGECACHE, pBuf, sz, N);
> 
> ```

参数 pBuf 是指向一段连续的字节范围的指针，SQLite 将用于页面缓存内存分配。该缓冲区的大小必须至少为 sz*N 字节。参数"sz"是每个页面缓存分配的大小。N 是可用分配的最大数量。

如果 SQLite 需要比"sz"字节大的页面缓存条目或者需要超过 N 个条目，它将回退到使用通用内存分配器。

## 3.3\. 前视内存分配器

SQLite 数据库连接 进行许多小型和短期的内存分配。这通常发生在使用 sqlite3_prepare_v2()编译 SQL 语句时，但也在运行 prepared statements 时（通过 sqlite3_step()）以较小程度发生。这些小型内存分配用于保存诸如表和列的名称、解析树节点、单个查询结果值和 B-树游标对象等内容。因此，会有许多 malloc()和 free()的调用 - 这么多的调用，以至于 malloc()和 free()最终使用了分配给 SQLite 的 CPU 时间的相当大一部分。

SQLite 版本 3.6.1（2008-08-06）引入了查找内存分配器，以帮助减少内存分配负载。在查找分配器中，每个数据库连接预分配一个单个大内存块（通常在 60 到 120 千字节之间），并将该块分割成小的固定大小的“插槽”，每个插槽大约在 100 到 1000 字节之间。这成为查找内存池。此后，与数据库连接相关的且不太大的内存分配将使用查找池中的一个插槽来满足，而不是调用通用内存分配器。较大的分配仍然使用通用内存分配器，以及在所有查找池插槽都被检出时的分配。但在许多情况下，内存分配足够小且未使用的插槽很少，因此新的内存请求可以从查找池中满足。

因为查找分配（lookaside allocations）始终是相同大小，分配和释放算法非常快速。无需合并相邻的空闲插槽或搜索特定大小的插槽。每个数据库连接维护一个未使用插槽的单链表。分配请求只需取出该列表的第一个元素。释放操作只需将元素推送回列表的前端。此外，假定每个数据库连接已经在单个线程中运行（已经有互斥锁来强制执行此操作），因此不需要额外的互斥来序列化对查找插槽空闲列表的访问。因此，查找内存分配和释放非常快速。在 Linux 和 Mac OS X 工作站上的速度测试中，SQLite 显示出高达 10%和 15%的整体性能改进，具体取决于工作负载及查找的配置方式。

查找存储池的大小有一个全局默认值，但也可以根据每个连接进行配置。要在编译时更改查找存储池的默认大小，请使用-DSQLITE_DEFAULT_LOOKASIDE=*SZ,N*选项。要在启动时更改查找存储池的默认大小，请使用 sqlite3_config()接口：

> ```sql
> sqlite3_config(SQLITE_CONFIG_LOOKASIDE, sz, cnt);
> 
> ```

参数“sz”是每个查找槽的字节大小。“cnt”参数是每个数据库连接的总查找存储池槽数量。分配给每个数据库连接的总查找存储池内存量为 sz*cnt 字节。

可以使用以下调用为单个数据库连接“db”更改查找存储池：

> ```sql
> sqlite3_db_config(db, SQLITE_DBCONFIG_LOOKASIDE, pBuf, sz, cnt);
> 
> ```

参数“pBuf”是指用于查找存储池的内存空间的指针。如果 pBuf 为 NULL，则 SQLite 将使用 sqlite3_malloc()获取自己的内存池空间。参数“sz”和“cnt”分别是每个查找槽的大小和槽数量。如果 pBuf 不为 NULL，则它必须指向至少 sz*cnt 字节的内存。

查找存储池的配置只能在数据库连接没有未完成的查找分配时更改。因此，在使用 sqlite3_open()（或等效函数）创建数据库连接后立即设置配置，并在连接上评估任何 SQL 语句之前设置。

### 3.3.1\. 双大小查找存储池

从 SQLite 版本 3.31.0（2020-01-22）开始，Lookaside 支持两个内存池，每个内存池都有不同大小的槽。小槽池使用 128 字节的槽，而大槽池使用由 SQLITE_DBCONFIG_LOOKASIDE 指定的大小（默认为 1200 字节）。这样分成两个池允许更频繁地使用 Lookaside 来进行内存分配，同时将每个数据库连接的堆使用量从 120KB 降低到 48KB。

配置继续使用 SQLITE_DBCONFIG_LOOKASIDE 或 SQLITE_CONFIG_LOOKASIDE 配置选项，如上所述，带有参数"sz"和"cnt"。用于 Lookaside 的总堆空间继续为 sz*cnt 字节。但是这个空间被分配给小槽 Lookaside 和大槽 Lookaside，优先考虑小槽 Lookaside。槽的总数通常会超过"cnt"，因为"sz"通常比 128 字节的小槽大小要大得多。

默认的 Lookaside 配置已经从每个 1200 字节的 100 个槽（120KB）变为每个 1200 字节的 40 个槽（48KB）。这些空间最终被分配为每个 128 字节的 93 个槽和每个 1200 字节的 30 个槽。因此，更多的 Lookaside 槽是可用的，但使用的堆空间要少得多。

默认的 Lookaside 配置、小槽的大小以及如何在小槽和大槽之间分配堆空间的细节，都可能在一个版本发布后发生变化。

## 3.4\. 内存状态

默认情况下，SQLite 会记录其内存使用情况的统计信息。这些统计信息有助于确定应用程序实际需要多少内存。这些统计数据还可以用于高可靠性系统，以确定内存使用是否接近或超过 Robson 证明的限制，从而使得内存分配子系统有可能崩溃。

大多数内存统计信息是全局的，因此必须使用互斥锁对统计信息进行串行化跟踪。默认情况下启用统计信息，但存在一个选项可禁用它们。通过禁用内存统计信息，在每次内存分配和释放时避免进入和退出互斥锁。在互斥操作昂贵的系统上，这种节省是显著的。为了在启动时禁用内存统计信息，使用以下接口：

> ```sql
> sqlite3_config(SQLITE_CONFIG_MEMSTATUS, onoff);
> 
> ```

"onoff" 参数为 true 表示启用内存统计跟踪，false 表示禁用统计跟踪。

假设统计数据已启用，可以使用以下例程访问它们：

> ```sql
> sqlite3_status(verb, &current, &highwater, resetflag);
> 
> ```

"verb" 参数决定访问哪种统计信息。定义了各种动词。随着 sqlite3_status() 接口的成熟，预计列表将增长。所选参数的当前值写入整数 "current"，最高历史值写入整数 "highwater"。如果 resetflag 为 true，则在调用返回后，高水位标记会被重置为当前值。

使用不同的接口查找与单个数据库连接 关联的统计信息：

> ```sql
> sqlite3_db_status(db, verb, &current, &highwater, resetflag);
> 
> ```

此接口类似，但它将指向数据库连接 的指针作为其第一个参数，并返回关于该对象而不是整个 SQLite 库的统计信息。sqlite3_db_status() 接口当前仅识别单个动词 SQLITE_DBSTATUS_LOOKASIDE_USED，尽管未来可能会添加其他动词。

连接统计数据不使用全局变量，因此不需要互斥体来更新或访问。因此，即使关闭了 SQLITE_CONFIG_MEMSTATUS，连接统计数据仍然可以正常工作。

## 3.5\. 设置内存使用限制

sqlite3_soft_heap_limit64()接口可用于设置 SQLite 通用内存分配器在任一时间允许的总未完成内存的上限。如果试图分配超出软堆限制指定的内存，则 SQLite 将首先尝试释放缓存内存，然后继续分配请求。软堆限制机制仅在启用内存统计时有效，并且在编译 SQLite 库时最好启用 SQLITE_ENABLE_MEMORY_MANAGEMENT 选项。

在这里，“软”软堆限制的意义是：如果 SQLite 无法释放足够的辅助内存以保持在限制以下，则继续分配额外内存并超过其限制。这是基于使用额外内存比直接失败更好的理论。

自 SQLite 版本 3.6.1（2008-08-06）起，软堆限制仅适用于通用内存分配器。软堆限制不了解或与页缓存内存分配器或预留内存分配器交互。这个不足可能在将来的版本中得到解决。

# 4\. 防止内存分配失败的数学保证

动态内存分配问题，特别是内存分配器崩溃问题，已由 J. M. Robson 研究，并发布为：

> J. M. Robson. "关于动态存储分配的一些函数的界限". *计算机协会期刊*, 第 21 卷，第 8 期，1974 年 7 月，第 491-499 页。

让我们使用以下符号（与 Robson 的符号类似但不完全相同）：

> | **N** | 内存分配系统所需的原始内存量，以确保永远不会发生内存分配失败。 |
> | --- | --- |
> | **M** | 应用程序在任何时间点上曾经检查过的最大内存量。 |
> | **n** | 最大内存分配与最小内存分配的比率。我们假设每个内存分配大小都是最小内存分配大小的整数倍数。 |

Robson 证明了以下结果：

> **N** = **M***(1 + (log[2] **n**)/2) - **n** + 1

通俗地说，Robson 的证明表明，为了保证无故障运行，任何内存分配器必须使用大小为**N**的内存池，该大小超过了通过**n**，即最大和最小分配大小的比率，确定的最大内存使用量**M**的倍数。换句话说，除非所有内存分配的大小完全相同（**n**=1），否则系统需要比其一次使用的内存更多。此外，我们看到所需的多余内存量随着最大和最小分配之间的比率增加而迅速增长，因此有强烈的动机尽可能保持所有分配大小接近。

Robson 的证明是建设性的。他提供了一个算法，用于计算一系列分配和释放操作，如果可用内存比**N**少一字节，则会导致内存碎片化而导致分配失败。此外，Robson 还表明，类似于 memsys5 实现的二次幂首次适配内存分配器将不会因可用内存为**N**或更多字节而导致内存分配失败。

值 **M** 和 **n** 是应用程序的属性。如果应用程序构建方式使得 **M** 和 **n** 已知，或者至少有已知的上限，并且应用程序使用 memsys5 内存分配器并提供了 **N** 字节的可用内存空间使用 SQLITE_CONFIG_HEAP，则 Robson 证明在应用程序内部任何内存分配请求都不会失败。换句话说，应用程序开发者可以选择一个 **N** 的值，可以保证任何 SQLite 接口的调用永远不会返回 SQLITE_NOMEM。内存池永远不会变得如此碎片化，以至于无法满足新的内存分配请求。这对于软件故障可能导致损害、身体伤害或不可替代数据丢失的应用程序是一个重要的特性。

## 4.1\. 计算和控制参数 **M** 和 **n**

Robson 证明分别适用于 SQLite 使用的每个内存分配器：

+   通用内存分配器 (memsys5)。

+   页面缓存内存分配器。

+   旁路内存分配器。

对于除了 memsys5 之外的分配器，所有内存分配大小都相同。因此，**n**=1，因此 **N**=**M**。换句话说，内存池的大小不需要大于任何给定时刻使用的最大内存量。

在 SQLite 版本 3.6.1 中，页面缓存内存的使用相对较难控制，尽管计划在后续版本中引入机制，使页面缓存内存的控制变得更加容易。在引入这些新机制之前，唯一控制页面缓存内存的方法是使用 cache_size pragma。

对于安全关键应用程序，通常希望修改默认的旁路内存配置，以便在调用 sqlite3_open()时分配的初始旁路内存缓冲区不要太大，以至于强制**n**参数过大。为了控制**n**的大小，最好尝试将最大内存分配保持在 2 或 4 千字节以下。因此，旁路内存分配器的合理默认设置可以是以下任何一种：

> ```sql
> sqlite3_config(SQLITE_CONFIG_LOOKASIDE, 32, 32);  /* 1K */
> sqlite3_config(SQLITE_CONFIG_LOOKASIDE, 64, 32);  /* 2K */
> sqlite3_config(SQLITE_CONFIG_LOOKASIDE, 32, 64);  /* 2K */
> sqlite3_config(SQLITE_CONFIG_LOOKASIDE, 64, 64);  /* 4K */
> 
> ```

另一种方法是最初禁用旁路内存分配器：

> ```sql
> sqlite3_config(SQLITE_CONFIG_LOOKASIDE, 0, 0);
> 
> ```

然后，让应用程序维护一个单独的较大旁路内存缓冲池，它可以在创建时分配给数据库连接。在通常情况下，应用程序只会有一个单独的数据库连接，因此旁路内存池可以由一个单独的大缓冲区组成。

> ```sql
> sqlite3_db_config(db, SQLITE_DBCONFIG_LOOKASIDE, aStatic, 256, 500);
> 
> ```

旁路内存分配器实际上是作为性能优化而不是作为确保无故障内存分配的方法，因此对于安全关键操作完全禁用旁路内存分配器并不是不合理的选择。

通用目的的内存分配器是最难管理的内存池，因为它支持各种大小的分配。由于**n**是**M**的一个乘数，我们希望尽可能将**n**保持小。这意味着应该尽可能将 memsys5 的最小分配大小设置为尽可能大。在大多数应用中，旁路内存分配器能够处理小的分配。因此，将 memsys5 的最小分配大小设置为旁路分配的最大大小的 2、4 甚至 8 倍是合理的。512 的最小分配大小是一个合理的设置。

进一步保持**n**的小值，希望控制最大内存分配的大小。通用内存分配器的大请求可能来自几个源：

1.  包含大字符串或 BLOB 的 SQL 表行。

1.  复杂的 SQL 查询最终编译成大型预处理语句。

1.  内部使用的 SQL 解析器对象由 sqlite3_prepare_v2()调用。

1.  数据库连接对象的存储空间。

1.  页面缓存内存分配超出通用内存分配器。

1.  新数据库连接的看边缓存分配。

最后两个分配可以通过适当配置页面缓存内存分配器和看边内存分配器来控制和/或消除，如上所述。数据库连接对象所需的存储空间在某种程度上取决于数据库文件名的长度，但在 32 位系统上很少超过 2KB。（64 位系统由于指针大小增加，需要更多空间。）每个解析器对象使用约 1.6KB 的内存。因此，可以轻松控制上述第 3 至 6 个元素，以保持最大内存分配大小在 2KB 以下。

如果应用程序设计为管理小数据片段，则数据库不应包含任何大字符串或 BLOB，因此上述第 1 点不应成为问题。如果数据库确实包含大字符串或 BLOB，则应使用增量 BLOB I/O 读取它们，同时包含大字符串或 BLOB 的行不应通过除增量 BLOB I/O 以外的任何方式更新。否则，sqlite3_step()例程将在某些时候需要将整个行读入连续内存，这将涉及至少一个大内存分配。

大型内存分配的最终来源是用于保存编译复杂 SQL 操作生成的预处理语句所需的空间。SQLite 开发人员正在努力减少这里所需的空间量。但是大型复杂查询可能仍需要大小为几千字节的预处理语句。目前唯一的解决方法是让应用程序将复杂 SQL 操作分解成两个或两个以上简单操作，各自包含在单独的预处理语句中。

综上所述，应用程序通常应能将其最大内存分配大小保持在 2K 或 4K 以下。这给出了 log2 为 2 或 3 的值。这将限制 **N** 在 2 倍和 2.5 倍 **M** 之间。

应用程序所需的通用内存的最大量由诸如应用程序同时打开了多少个数据库连接和预处理语句对象，以及预处理语句的复杂性等因素确定。对于任何给定的应用程序，这些因素通常是固定的，可以使用 SQLITE_STATUS_MEMORY_USED 进行实验性确定。一个典型的应用程序可能只使用约 40KB 的通用内存。这给出了**N** 约为 100KB 的值。

## 4.2\. 韧性破坏

如果   如果 SQLite 内存分配子系统配置为无故障运行，但实际内存使用量超过了由罗布森证明设置的设计限制，SQLite 通常会继续正常运行。 页缓存内存分配器和查找内存分配器会自动切换到 memsys5 通用内存分配器。通常情况下，即使 **M** 和/或 **n** 超过罗布森证明所规定的限制，memsys5 内存分配器也会继续工作而不产生碎片。罗布森证明表明，在这种情况下，内存分配可能会发生故障，但这种故障需要一系列特别糟糕的分配和释放操作序列 - SQLite 从未观察到过这种序列。因此，在实践中，通常情况下，罗布森设置的限制可以大幅度超过而不会产生不良影响。

尽管如此，应用程序开发人员被告诫要监控内存分配子系统的状态，并在内存使用量接近或超过罗布森限制时发出警报。通过这种方式，应用程序将提前向操作员提供丰富的警告以避免失败。SQLite 的内存统计接口为应用程序提供了完成此任务监控部分所需的所有机制。

# 5\. 内存接口的稳定性

**更新：** 截至 SQLite 版本 3.7.0（2010-07-21），所有 SQLite 内存分配接口都被认为是稳定的，并将在未来的版本中得到支持。
