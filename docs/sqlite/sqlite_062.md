# 1\. 概述

> 原文：[`sqlite.com/wal.html`](https://sqlite.com/wal.html)

SQLite 实现原子提交和回滚的默认方法是回滚日志。从版本 3.7.0（2010-07-21）开始，提供了一种新的“写前日志”选项（以下简称 WAL）。

使用 WAL 而不是回滚日志的优缺点有：

1.  在大多数场景下，WAL 明显更快。

1.  WAL 提供更多的并发性，因为读者不会阻塞写入者，写入者也不会阻塞读者。读取和写入可以并发进行。

1.  使用 WAL 时，磁盘 I/O 操作倾向于更加顺序化。

1.  WAL 使用较少的 fsync()操作，因此在 fsync()系统调用存在问题的系统上，它更不容易出现问题。

但是也有一些缺点：

1.  WAL 通常要求 VFS 支持共享内存原语。（例外情况：无共享内存的 WAL）内置的 unix 和 windows VFS 支持此功能，但第三方扩展的自定义操作系统的 VFS 可能不支持。

1.  使用数据库的所有进程必须在同一台主机上；WAL 不适用于网络文件系统。

1.  对涉及多个 ATTACH 数据库的事务在每个单独的数据库中是原子的，但在整个数据库集合中不是原子的。

1.  进入 WAL 模式后，无法通过 VACUUM 或使用备份 API 从空数据库或备份中恢复来更改 page_size。必须处于回滚日志模式才能更改页面大小。

1.  从版本 3.22.0（2018-01-22）开始，如果`-shm`和`-wal`文件已经存在，或者这些文件可以被创建，或者数据库是不可变的，可以打开只读 WAL 模式数据库文件。

1.  在大多数情况下，WAL 可能会比传统的回滚日志方法稍微慢一些（可能慢 1%到 2%），特别是在主要进行读取而很少写入的应用程序中。

1.  每个数据库都有一个额外的准持久化的`-wal`文件和`-shm`共享内存文件，这可能会使 SQLite 在作为应用程序文件格式时不太适用。

1.  还有一个额外的检查点操作，虽然默认情况下是自动的，但应用程序开发人员仍然需要注意。

1.  从 version 3.11.0 (2016-02-15) 起，WAL 模式对大事务的处理效率与回滚模式一样高效，甚至更高。

# 2\. WAL 工作原理

传统的回滚日志通过将原始未更改的数据库内容复制到单独的回滚日志文件中，然后直接将更改写入数据库文件来工作。在崩溃或 ROLLBACK 的情况下，回滚日志中包含的原始内容将被播放回数据库文件，以将数据库文件恢复到其原始状态。当删除回滚日志时，会发生 COMMIT。

WAL 方法正好相反。原始内容保留在数据库文件中，而更改则附加到单独的 WAL 文件中。在 WAL 中附加一个特殊的记录指示提交时，就会发生 COMMIT。因此，COMMIT 可以在不写入原始数据库的情况下发生，这使得读者可以继续从未更改的原始数据库中操作，同时将更改提交到 WAL 中。多个事务可以附加到单个 WAL 文件的末尾。

## 2.1\. 检查点

当然，最终希望将附加在 WAL 文件中的所有事务转移到原始数据库中。将 WAL 文件事务移回数据库的过程称为“*检查点*”。

另一种理解回滚日志和写前日志之间差异的方法是，在回滚日志方法中，有两种基本操作，即读取和写入；而在写前日志方法中，则有三种基本操作：读取、写入和检查点。

默认情况下，当 WAL 文件达到 1000 页的阈值时，SQLite 会自动进行检查点。可以通过 SQLITE_DEFAULT_WAL_AUTOCHECKPOINT 编译时选项来指定不同的默认值。使用 WAL 的应用程序无需采取任何措施即可执行这些检查点。但如果需要的话，应用程序可以调整自动检查点的阈值。或者可以关闭自动检查点，在空闲时段或在单独的线程或进程中运行检查点。

## 2.2\. 并发性

当在 WAL 模式数据库上开始读取操作时，首先记住 WAL 中最后一个有效提交记录的位置。将这一点称为“结束标记”。因为 WAL 可以在各种读取器连接到数据库时不断增长并添加新的提交记录，每个读取器可能都有自己的结束标记。但对于任何特定的读取器，在事务期间结束标记保持不变，从而确保单个读取事务只能看到数据库内容在某个时间点存在的样子。

当读取器需要内容页时，首先检查 WAL 文件，看看是否有该页，如果有，则拉取在读取器结束标记之前 WAL 中出现的该页的最后一份拷贝。如果在读取器结束标记之前的 WAL 中不存在该页的任何拷贝，则从原始数据库文件读取该页。读取器可以存在于不同的进程中，为了避免强制每个读取器扫描整个 WAL 以查找页面（根据检查点运行频率，WAL 文件可能会增长到多兆字节），维护一种称为“wal-index”的数据结构在共享内存中，帮助读取器快速定位 WAL 中的页面，并最小化 I/O 操作。wal-index 大大提高了读取器的性能，但使用共享内存意味着所有读取器必须存在于同一台机器上。这就是为什么写前日志实现在网络文件系统上无法工作的原因。

写入者只是将新内容追加到 WAL 文件的末尾。由于写入者不会干扰读取操作的动作，因此写入者和读取者可以同时运行。但是，由于只有一个 WAL 文件，因此一次只能有一个写入者。

一个检查点操作将内容从 WAL 文件传输回原始数据库文件。检查点可以与读取操作同时运行，但是当检查点达到 WAL 中任何当前读取器结束标记之后的页面时，必须停止。检查点必须在那一点停止，否则可能会覆盖读取器正在使用的数据库文件的部分内容。检查点会在 wal-index 中记住其进展的程度，并在下一次调用时从上次停止的地方继续将 WAL 中的内容传输到数据库中。

因此，长时间运行的读取事务可能会阻止检查点程序取得进展。但是可以假定每个读取事务最终会结束，检查点程序就能够继续运行。

每当发生写入操作时，写入者会检查检查点程序的进展情况，如果整个 WAL 已经传输到数据库并同步，且没有读取器在使用 WAL，那么写入者将回溯 WAL 到开始位置，并开始将新事务放在 WAL 的开头。这种机制防止 WAL 文件无限增长。

## 2.3\. 性能考虑

写事务非常快，因为它们只涉及写入内容一次（与回滚日志事务相比需要写两次），并且所有写入都是顺序的。此外，只要应用程序愿意在断电或硬重启后牺牲耐久性，就不需要将内容同步到磁盘。（如果 PRAGMA synchronous 设置为 FULL，写入者在每次事务提交时都会同步 WAL，但如果设置为 NORMAL，则会省略此同步。）

另一方面，随着 WAL 文件大小的增长，读取性能会下降，因为每个读取器必须检查 WAL 文件的内容，并且检查 WAL 文件所需的时间与 WAL 文件的大小成正比。Wal-index 能够更快地在 WAL 文件中找到内容，但随着 WAL 文件大小的增加，性能仍会下降。因此，为了保持良好的读取性能，通过定期运行检查点来保持 WAL 文件大小很重要。

Checkpointing 在避免因断电或硬重启后可能导致数据库损坏的情况下确实需要同步操作。在将 WAL 中的内容移入数据库之前，必须将 WAL 同步到持久存储，而且在重置 WAL 之前必须同步数据库文件。Checkpoint 还需要更多的寻址操作。Checkpointer 尽力将尽可能多的数据库顺序页面写入（页面按升序从 WAL 转移到数据库），但即便如此，在页面写入中通常仍会有许多寻址操作。这些因素结合在一起使得检查点比写事务慢。

默认策略是允许连续的写事务增长 WAL，直到 WAL 大约为 1000 页大小，然后为每个后续的 COMMIT 运行一个检查点操作，直到 WAL 被重置为小于 1000 页。默认情况下，检查点将由执行将 WAL 推到其大小限制的 COMMIT 的同一线程自动运行。这导致大多数 COMMIT 操作非常快，但偶尔会有一些 COMMIT（触发检查点的那些）变得慢得多。如果这种效果不理想，应用程序可以禁用自动检查点并在单独的线程或进程中运行周期性检查点。 （可以查看下文显示的命令和接口来完成此操作。）

请注意，当 PRAGMA synchronous 设置为 NORMAL 时，检查点是唯一会发出 I/O 屏障或同步操作（在 Unix 上为 fsync()，在 Windows 上为 FlushFileBuffers()）的操作。因此，如果应用程序在单独的线程或进程中运行检查点，则执行数据库查询和更新的主线程或进程将不会因同步操作而阻塞。这有助于防止在繁忙磁盘驱动器上运行的应用程序中出现 "锁死"。这种配置的缺点是事务不再是持久的，可能会在断电或硬重置后回滚。

同时请注意，平均读取性能和平均写入性能之间存在权衡。为了最大化读取性能，应尽量保持 WAL 的大小尽可能小，并且频繁运行检查点，可能是每次 COMMIT 都运行一次。为了最大化写入性能，应尽可能将每次检查点的成本分摊到尽可能多的写入中，意味着应该不经常运行检查点，并允许 WAL 在每次检查点之前尽可能增大。因此，根据应用程序的相对读写性能需求，运行检查点的频率决策可能因应用程序而异。默认策略是一旦 WAL 达到 1000 页就运行一次检查点，在工作站上的测试应用程序中，这种策略似乎运行良好，但在不同平台或不同工作负载下，可能有其他更好的策略。

# 3\. 激活和配置 WAL 模式

默认情况下，SQLite 数据库连接使用 journal_mode=DELETE。要转换为 WAL 模式，请使用以下 pragma：

> ```sql
> PRAGMA journal_mode=WAL;
> 
> ```

`journal_mode` pragma 返回一个字符串，表示新的日志模式。成功时，该 pragma 将返回字符串"`wal`"。如果无法完成转换到 WAL（例如，如果 VFS 不支持必要的共享内存原语），则日志模式将保持不变，并且从原语返回的字符串将是先前的日志模式（例如"`delete`"）。

## 3.1\. 自动检查点

默认情况下，当 WAL 文件的大小达到 1000 页或更多，或者当数据库文件上的最后一个数据库连接关闭时，SQLite 将自动进行检查点。默认配置旨在适用于大多数应用程序。但是需要更多控制的程序可以使用 wal_checkpoint pragma 或调用 C 接口 sqlite3_wal_checkpoint()来强制进行检查点。可以通过使用 wal_autocheckpoint pragma 或调用 C 接口 sqlite3_wal_autocheckpoint()来更改自动检查点阈值或完全禁用自动检查点。程序还可以使用 sqlite3_wal_hook()来注册在任何事务提交到 WAL 时调用的回调。然后，这个回调可以根据自己认为适当的标准调用 sqlite3_wal_checkpoint()或 sqlite3_wal_checkpoint_v2()。 （自动检查点机制是围绕 sqlite3_wal_hook()简单封装的。）

## 3.2\. 应用启动的检查点

应用程序可以通过对数据库上任何可写数据库连接调用 sqlite3_wal_checkpoint()或 sqlite3_wal_checkpoint_v2()来启动检查点。检查点有三种类型，根据它们的侵略性而有所不同：PASSIVE、FULL 和 RESTART。默认的检查点样式是 PASSIVE，它尽可能多地进行工作，而不干扰其他数据库连接，如果存在并发的读取或写入，可能无法完成。由 sqlite3_wal_checkpoint()和自动检查点机制启动的所有检查点都是 PASSIVE。FULL 和 RESTART 检查点尝试更努力地运行检查点以完成，并且只能通过调用 sqlite3_wal_checkpoint_v2()来启动。有关 FULL 和 RESET 检查点的附加信息，请参阅 sqlite3_wal_checkpoint_v2()文档。

## 3.3\. WAL 模式的持久性

与其他日志模式不同，PRAGMA journal_mode=WAL 是持久的。如果一个进程设置了 WAL 模式，然后关闭并重新打开数据库，数据库将以 WAL 模式重新启动。相比之下，如果一个进程设置（例如）PRAGMA journal_mode=TRUNCATE，然后关闭并重新打开数据库，数据库将以默认的回滚模式 DELETE 而不是前一个 TRUNCATE 设置重新启动。

WAL 模式的持久性意味着可以将应用程序转换为使用 SQLite 的 WAL 模式，而无需对应用程序本身进行任何更改。只需在使用命令行 shell 或其他实用程序的数据库文件上运行"`PRAGMA journal_mode=WAL;`"，然后重新启动应用程序即可。

如果在任何一个连接上设置了 WAL（Write-Ahead Logging）日志模式，那么同一数据库文件上的所有连接都将设置为 WAL 日志模式。

# 4\. WAL 文件

当在 WAL 模式数据库上打开数据库连接时，SQLite 会维护一个额外的日志文件，称为“Write Ahead Log”或“WAL 文件”。此文件在磁盘上的名称通常是数据库文件名加上额外的“`-wal`”后缀，尽管如果 SQLite 使用 SQLITE_ENABLE_8_3_NAMES 编译，则可能适用不同的命名规则。

只要有任何数据库连接打开数据库，WAL 文件就会存在。通常，当最后一个连接关闭时，WAL 文件会被自动删除。但是，如果最后一个持有数据库打开连接的进程退出时没有正确关闭数据库连接，或者使用 SQLITE_FCNTL_PERSIST_WAL 文件控制，那么在关闭数据库的所有连接后，WAL 文件可能会在磁盘上保留。WAL 文件是数据库的持久状态的一部分，如果复制或移动数据库，应将其与数据库一起保存。如果将数据库文件与其 WAL 文件分开，则先前提交到数据库的事务可能会丢失，或者数据库文件可能会损坏。删除 WAL 文件的唯一安全方法是使用其中一个 sqlite3_open()接口打开数据库文件，然后立即使用 sqlite3_close()关闭数据库。

WAL 文件格式有明确定义，是跨平台的。

# 5\. 只读数据库

旧版本的 SQLite 无法读取只读的 WAL 模式数据库。换句话说，需要写访问权限才能读取 WAL 模式数据库。从 SQLite 版本 3.22.0（2018-01-22）开始，这个约束条件被放宽。

在较新版本的 SQLite 上，只要满足以下一个或多个条件，就可以读取只读介质上的 WAL 模式数据库，或者没有写权限的 WAL 模式数据库：

1.  `-shm`和`-wal`文件已经存在且可读

1.  包含数据库的目录具有写权限，以便创建`-shm`和`-wal`文件。

1.  使用不可变查询参数打开数据库连接。

即使可以打开一个只读的 WAL 模式数据库，最好在将 SQLite 数据库映像刻录到只读介质之前转换为 PRAGMA journal_mode=DELETE。

# 6\. 避免过大的 WAL 文件

在正常情况下，新内容会追加到 WAL 文件，直到 WAL 文件累积约 1000 页（大约 4MB 大小），此时将自动运行检查点并回收 WAL 文件。检查点通常不会截断 WAL 文件（除非设置了 journal_size_limit pragma）。相反，它只是导致 SQLite 从头开始覆写 WAL 文件。这样做是因为覆写现有文件通常比追加更快。当数据库的最后一个连接关闭时，该连接会进行最后一个检查点，然后删除 WAL 及其关联的共享内存文件，以清理磁盘。

因此，在绝大多数情况下，应用程序根本不需要担心 WAL 文件。SQLite 将自动处理它。但是可能会使 SQLite 进入 WAL 文件无限增长的状态，导致磁盘空间使用过多和查询速度变慢。以下几点列举了可能发生这种情况的一些方式以及如何避免它们。

+   **禁用自动检查点机制。** 在默认配置中，SQLite 在 WAL 文件超过 1000 页时，在任何事务结束时会进行检查点。然而，存在编译时和运行时选项可以禁用或推迟此自动检查点。如果应用程序禁用了自动检查点，则没有任何机制可以防止 WAL 文件过度增长。

+   **检查点饥饿。** 只有在没有其他数据库连接使用 WAL 文件时，检查点才能完成运行并重置 WAL 文件。如果另一个连接有打开的读事务，则检查点无法重置 WAL 文件，因为这样做可能会在读取者正在使用时删除内容。检查点会尽可能多地工作而不会打扰读取者，但无法完成运行。在下一次写事务之后，检查点会从中断的地方重新启动。这种过程会重复，直到某个检查点能够完成为止。

    然而，如果一个数据库有许多并发的重叠读取者，并且始终至少有一个活动读取者，则没有检查点能够完成，因此 WAL 文件会无限增长。

    可通过确保存在“读者间隙”来避免此情况：即没有进程从数据库读取，并尝试在这些时间进行检查点。在具有许多并发读取器的应用程序中，还可以考虑使用 SQLITE_CHECKPOINT_RESTART 或 SQLITE_CHECKPOINT_TRUNCATE 选项手动运行检查点，这将确保检查点在返回之前完成运行。使用 SQLITE_CHECKPOINT_RESTART 和 SQLITE_CHECKPOINT_TRUNCATE 的缺点是读取器在检查点运行时可能会阻塞。

+   **非常大的写事务。** 只有当没有其他事务运行时，检查点才能完成，这意味着无法在写事务的中间重置 WAL 文件。因此，对大型数据库的大更改可能导致 WAL 文件变得很大。在写事务完成后（假设没有其他读取器阻塞），WAL 文件将被检查点。但与此同时，文件可能会变得非常大。

    截至 SQLite 版本 3.11.0（2016-02-15），单个事务的 WAL 文件大小应与事务本身成比例。只有被事务更改的页面才会写入 WAL 文件一次。但是，对于较旧版本的 SQLite，如果事务比页面缓存大，同一页面可能会多次写入 WAL 文件。

# 7\. WAL-Index 的共享内存实现

wal-index 是通过映射成为鲁棒性普通文件来实现的。早期（预发布）的 WAL 模式实现将 wal-index 存储在易失性共享内存中，例如 Linux 上创建在/dev/shm 或其他 Unix 系统上的/tmp 文件。该方法的问题在于，使用不同根目录（通过[chroot](http://en.wikipedia.org/wiki/Chroot)修改）的进程将看到不同的文件，从而使用不同的共享内存区域，导致数据库损坏。在各种 Unix 系统中，其他创建无名称共享内存块的方法并不可移植。我们也找不到在 Windows 上创建无名称共享内存块的方法。确保所有访问同一数据库文件的进程使用相同共享内存的唯一方法是通过在与数据库本身相同目录中映射文件来创建共享内存。

使用普通磁盘文件提供共享内存的缺点是，它可能会通过将共享内存写入磁盘而进行不必要的磁盘 I/O。但是，开发人员认为这不是一个主要问题，因为 wal-index 很少超过 32 KiB，并且从未同步。此外，当最后一个数据库连接断开时，通常会删除 wal-index 的后备文件，从而通常防止发生任何真正的磁盘 I/O。

对于默认的共享内存实现不可接受的专用应用程序可以通过自定义的 VFS 设计替代方法。例如，如果已知特定数据库仅由单个进程内的线程访问，那么可以使用堆内存而不是真正的共享内存来实现 wal-index。

# 8\. 在没有共享内存的情况下使用 WAL

从 SQLite 版本 3.7.4（2010-12-07）开始，即使没有共享内存，只要在第一次尝试访问之前将 locking_mode 设置为 EXCLUSIVE，也可以创建、读取和写入 WAL 数据库。换句话说，如果确保只有一个进程访问数据库，就可以使用不支持 "version 2" 共享内存方法 xShmMap、xShmLock、xShmBarrier 和 xShmUnmap 的传统 VFSes 创建、读取和写入 WAL 数据库。

如果在第一次访问 WAL 模式数据库之前设置了 EXCLUSIVE 锁定模式，那么 SQLite 永远不会尝试调用任何共享内存方法，因此永远不会创建共享内存的 wal-index。在这种情况下，只要日志模式为 WAL，数据库连接将保持在 EXCLUSIVE 模式；尝试使用 "`PRAGMA locking_mode=NORMAL;`" 修改锁定模式将不起作用。退出 EXCLUSIVE 锁定模式的唯一方法是首先退出 WAL 日志模式。

如果第一次访问 WAL 模式数据库时处于 NORMAL 锁定模式，则会创建共享内存的 wal-index。这意味着底层 VFS 必须支持 "version 2" 共享内存。如果 VFS 不支持共享内存方法，则尝试打开已经处于 WAL 模式的数据库或尝试将数据库转换为 WAL 模式将失败。只要仅有一个连接使用共享内存的 wal-index，就可以在 NORMAL 和 EXCLUSIVE 之间自由更改锁定模式。只有在省略共享内存的 wal-index，即在第一次访问 WAL 模式数据库之前，锁定模式才会被固定为 EXCLUSIVE。

# 9\. 有时在 WAL 模式下查询会返回 SQLITE_BUSY

WAL 模式的 第二个优势 是写操作不会阻塞读操作，读操作也不会阻塞写操作。这基本上是正确的。但在某些罕见的情况下，对 WAL 模式数据库的查询可能会返回 SQLITE_BUSY，因此应用程序应对此种情况做好准备。

查询 WAL 模式数据库可能返回 SQLITE_BUSY 的情况包括以下几种：

+   如果另一个数据库连接以 排他锁定模式 打开数据库模式，则针对数据库的所有查询都将返回 SQLITE_BUSY。Chrome 和 Firefox 都以排他锁定模式打开它们的数据库文件，因此在应用程序运行时尝试读取 Chrome 或 Firefox 数据库将遇到此问题，例如。

+   当最后一个连接到特定数据库正在关闭时，该连接将在清理 WAL 和共享内存文件时短暂获取独占锁。如果在第一个连接仍在其清理过程中时尝试打开和查询数据库，则第二个连接可能会收到 SQLITE_BUSY 错误。

+   如果最后一个连接到数据库崩溃，则打开数据库的第一个新连接将启动恢复过程。在恢复期间保持独占锁。因此，如果第三个数据库连接在第二个连接正在运行恢复时尝试插入和查询，第三个连接将收到 SQLITE_BUSY 错误。

# 10\. 向后兼容性

数据库文件格式在 WAL 模式下保持不变。但是，WAL 文件和 wal-index 是新概念，因此旧版本的 SQLite 不会知道如何恢复在发生崩溃时正在 WAL 模式下运行的 SQLite 数据库。为了防止较旧的 SQLite 版本（2010-07-22 之前的版本，即 3.7.0 之前）尝试恢复 WAL 模式数据库（并使情况变得更糟），数据库文件格式版本号（数据库头部的第 18 和 19 字节）在 WAL 模式下从 1 增加到 2。因此，如果较旧版本的 SQLite 尝试连接正在 WAL 模式下运行的 SQLite 数据库，它将报告类似于 "文件已加密或不是数据库" 的错误。

可以使用如下的 pragma 明确退出 WAL 模式：

> ```sql
> PRAGMA journal_mode=DELETE;
> 
> ```

故意退出 WAL 模式会将数据库文件格式版本号更改回 1，以便较旧版本的 SQLite 再次访问数据库文件。