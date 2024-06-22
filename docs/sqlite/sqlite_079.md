# 内存数据库

> 原文：[`sqlite.com/inmemorydb.html`](https://sqlite.com/inmemorydb.html)

通常情况下，SQLite 数据库存储在单个普通磁盘文件中。但是，在某些情况下，数据库可能存储在内存中。

强制使 SQLite 数据库完全存在于内存中最常见的方法是使用特殊文件名 "**:memory:**" 打开数据库。换句话说，不要向 sqlite3_open()，sqlite3_open16()，或者 sqlite3_open_v2() 函数传递一个真实的磁盘文件名，而是传递字符串 ":memory:"。例如：

> ```sql
> rc = sqlite3_open(":memory:", &db);
> 
> ```

当这样做时，不会打开任何磁盘文件。相反，会在内存中纯粹创建一个新数据库。只要数据库连接关闭，数据库就会停止存在。每个 :memory: 数据库都与其他每个数据库不同。因此，使用 ":memory:" 文件名分别打开两个数据库连接将创建两个独立的内存数据库。

特殊文件名 ":memory:" 可以在允许使用数据库文件名的任何地方使用。例如，可以作为 ATTACH 命令中的 *filename*：

> ```sql
> ATTACH DATABASE ':memory:' AS aux1;
> 
> ```

注意，为了使特殊 ":memory:" 名称生效并创建纯内存数据库，文件名中不能包含其他附加文本。因此，可以通过在文件名前加上路径名来创建基于磁盘的数据库，例如："./:memory:"。

特殊 ":memory:" 文件名在使用 URI filenames 时也适用。例如：

> ```sql
> rc = sqlite3_open("file::memory:", &db);
> 
> ```

或者，

> ```sql
> ATTACH DATABASE 'file::memory:' AS aux1;
> 
> ```

## 内存数据库和共享缓存

如果使用 URI filename 打开内存数据库，则允许使用 shared cache。如果使用未装饰的 ":memory:" 名称指定内存数据库，则该数据库始终具有私有缓存，并且仅对最初打开它的数据库连接可见。但是，同一个内存数据库可以通过两个或更多数据库连接打开，如下所示：

> ```sql
> rc = sqlite3_open("file::memory:?cache=shared", &db);
> 
> ```

或者，

> ```sql
> ATTACH DATABASE 'file::memory:?cache=shared' AS aux1;
> 
> ```

这允许单独的数据库连接共享同一个内存数据库。当然，所有共享内存数据库的数据库连接都需要在同一个进程中。当最后一个连接到数据库关闭时，数据库将自动删除并释放内存。

如果在单个进程中需要两个或多个不同但可共享的内存数据库，则可以使用 mode=memory 查询参数，并使用 URI 文件名来创建命名的内存数据库：

> ```sql
> rc = sqlite3_open("file:memdb1?mode=memory&cache=shared", &db);
> 
> ```

或者，

> ```sql
> ATTACH DATABASE 'file:memdb1?mode=memory&cache=shared' AS aux1;
> 
> ```

当以这种方式命名内存数据库时，它将只与另一个使用完全相同名称的连接共享其缓存。

## 临时数据库

当传递给 sqlite3_open()或 ATTACH 的数据库文件名为空字符串时，将创建一个新的临时文件来保存数据库。

> ```sql
> rc = sqlite3_open("", &db);
> 
> ```
> 
> ```sql
> ATTACH DATABASE '' AS aux2;
> 
> ```

每次都会创建一个不同的临时文件，因此，就像特殊的":memory:"字符串一样，对临时数据库的两个数据库连接各自拥有自己的私有数据库。临时数据库在创建它们的连接关闭时会自动删除。

尽管为每个临时数据库分配了一个磁盘文件，但实际上临时数据库通常驻留在内存分页器缓存中，因此通过":memory:"创建的纯内存数据库与通过空文件名创建的临时数据库之间几乎没有区别。唯一的区别是":memory:"数据库必须始终驻留在内存中，而临时数据库的部分可能会在数据库变大或 SQLite 受到内存压力时刷新到磁盘。

前几段描述了在默认的 SQLite 配置下临时数据库的行为。如果需要，应用程序可以使用 temp_store pragma 和 SQLITE_TEMP_STORE 编译时参数来强制临时数据库像纯内存数据库一样运行。
