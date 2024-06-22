# 1\. 总结

> 原文：[`sqlite.com/autoinc.html`](https://sqlite.com/autoinc.html)

1.  如果没有严格需要，应避免使用 AUTOINCREMENT 关键字，因为它会增加额外的 CPU、内存、磁盘空间和磁盘 I/O 开销。通常情况下是不必要的。

1.  在 SQLite 中，类型为 `INTEGER PRIMARY KEY` 的列是 ROWID 的别名（在没有 ROWID 的表中除外），它始终是一个 64 位有符号整数。

1.  在 INSERT 中，如果未为 ROWID 或 INTEGER PRIMARY KEY 列明确指定值，则它将自动填充为未使用的整数，通常是当前使用的最大 ROWID 加一。这是无论是否使用 AUTOINCREMENT 关键字都成立。

1.  如果在 `INTEGER PRIMARY KEY` 之后出现 `AUTOINCREMENT` 关键字，则会更改自动分配 ROWID 的算法，以防止数据库生命周期内重新使用 ROWID。换句话说，AUTOINCREMENT 的目的是防止重新使用先前删除行的 ROWID。

# 2\. 背景

在 SQLite 中，表行通常具有唯一的 64 位有符号整数 ROWID（在 WITHOUT ROWID 表中除外）。

可以使用特殊列名 ROWID、_ROWID_ 或 OID 访问 SQLite 表的 ROWID。除非声明普通表列使用其中一个特殊名称，否则使用该名称将引用已声明的列而不是内部 ROWID。

如果表包含类型为 INTEGER PRIMARY KEY 的列，则该列成为 ROWID 的别名。然后，可以使用任何四个不同的名称访问 ROWID，即上述三个原始名称或给定给 INTEGER PRIMARY KEY 列的名称。所有这些名称都是彼此的别名，并且在任何上下文中均有效。

当向 SQLite 表插入新行时，可以在 INSERT 语句中指定 ROWID，也可以由数据库引擎自动分配。要手动指定 ROWID，只需将其包含在要插入的值列表中。例如：

```sql
CREATE TABLE test1(a INT, b TEXT);
INSERT INTO test1(rowid, a, b) VALUES(123, 5, 'hello');

```

如果在插入时未指定 ROWID，或者指定的 ROWID 值为 NULL，则会自动创建适当的 ROWID。通常的算法是在插入之前给新创建的行一个比表中最大 ROWID 大一的 ROWID。如果表最初为空，则使用 ROWID 1。如果最大 ROWID 等于最大可能的整数（9223372036854775807），则数据库引擎开始随机选择正候选 ROWID，直到找到一个尚未使用的。如果在合理的尝试次数后找不到未使用的 ROWID，则插入操作会因 SQLITE_FULL 错误而失败。如果没有显式插入负 ROWID 值，则自动生成的 ROWID 值始终大于零。

如上所述，普通的 ROWID 选择算法将生成单调递增的唯一 ROWID，只要您从未使用过最大 ROWID 值，也从未删除具有最大 ROWID 的表中的条目。如果删除行或创建具有最大可能 ROWID 的行，则在创建新行时可能会重新使用先前删除行的 ROWID，并且新创建的 ROWID 可能不是严格升序的。

# 3\. AUTOINCREMENT 关键字

如果列具有类型 INTEGER PRIMARY KEY AUTOINCREMENT，则使用稍有不同的 ROWID 选择算法。为新行选择的 ROWID 至少比此前同一表中已存在的最大 ROWID 大 1。如果表以前从未包含任何数据，则使用 ROWID 1。如果先前插入了最大可能的 ROWID，则不允许新的 INSERT，并且任何尝试插入新行都将因 SQLITE_FULL 错误而失败。仅考虑已提交的先前事务的 ROWID 值。已回滚的 ROWID 值将被忽略并可以重新使用。

SQLite 使用一个名为"sqlite_sequence"的内部表来跟踪最大的 ROWID。sqlite_sequence 表会在创建具有 AUTOINCREMENT 列的普通表时自动创建，如果该表尚不存在。第一次写入 AUTOINCREMENT 表时会创建与具有 AUTOINCREMENT 列的表对应的 sqlite_sequence 行，并在任何后续增加最大 ROWID 的写入操作时进行更新。可以使用普通的 UPDATE、INSERT 和 DELETE 语句修改 sqlite_sequence 表的内容。但在进行此类更改之前，请确保知道自己在做什么。sqlite_sequence 表不会跟踪与 UPDATE 语句相关的 ROWID 更改，只会跟踪 INSERT 语句。

使用 AUTOINCREMENT 关键字实现的行为与默认行为略有不同。使用 AUTOINCREMENT，自动选择的 ROWID 的行保证在同一数据库的同一表中从未使用过。而且自动生成的 ROWID 保证是单调递增的。这些在某些应用中是重要的属性。但如果您的应用程序不需要这些属性，您可能应该继续使用默认行为，因为使用 AUTOINCREMENT 需要在每次插入行时做额外的工作，从而导致插入操作稍慢一些。

注意，“单调递增”并不意味着 ROWID 总是增加恰好一个。通常情况下增加一个。但是，如果插入失败，例如由于唯一性约束，插入失败的 ROWID 可能不会在后续插入中重新使用，从而导致 ROWID 序列中出现间隙。AUTOINCREMENT 保证了自动选择的 ROWID 会递增，但不能保证它们会连续。

由于 AUTOINCREMENT 关键字改变了 ROWID 选择算法的行为，因此在 WITHOUT ROWID 表或除 INTEGER PRIMARY KEY 外的任何表列上都不允许使用 AUTOINCREMENT。任何试图在 WITHOUT ROWID 表或非 INTEGER PRIMARY KEY 列上使用 AUTOINCREMENT 的尝试都会导致错误。
