# 1\. 引言

> 原文：[`sqlite.com/optoverview.html`](https://sqlite.com/optoverview.html)

本文档概述了 SQLite 查询规划器和优化器的工作原理。

给定一个单个 SQL 语句，根据语句本身的复杂性和底层数据库模式的复杂性，可能会有几十个、几百个，甚至上千种实现该语句的方式。查询规划器的任务是选择最小化磁盘 I/O 和 CPU 开销的算法。

额外的背景信息可在索引教程文档中找到。下一代查询规划器文档详细描述了如何选择连接顺序。

# 2\. WHERE 子句分析

在分析之前，会进行以下转换，将所有连接约束移到 WHERE 子句中：

+   所有自然连接都转换为带有 USING 子句的连接。

+   所有 USING 子句（包括前面步骤创建的）都会转换为等效的 ON 子句。

+   所有 ON 子句（包括前面步骤创建的）都作为 WHERE 子句中的新连接项（AND 连接的术语）添加。

SQLite 在 WHERE 子句中出现的连接约束和内连接的 ON 子句中的约束之间没有区别，因为这个区别不影响结果。然而，对于外连接，ON 子句约束和 WHERE 子句约束之间存在差异。因此，当 SQLite 将来自外连接的 ON 子句约束移动到 WHERE 子句时，它会向抽象语法树（AST）添加特殊标记，指示该约束来自外连接及其来源。在纯 SQL 文本中无法添加这些标记。因此，SQL 输入必须在外连接上使用 ON 子句。但在内部 AST 中，所有约束都是 WHERE 子句的一部分，因为将所有内容放在一个地方简化了处理过程。

在所有约束已转移到 WHERE 子句之后，WHERE 子句被分解为多个联结（以下简称为“条件”）。换句话说，WHERE 子句被分解为由 AND 运算符分隔的片段。如果 WHERE 子句由 OR 运算符分隔的约束（析取式）组成，则整个子句被视为单个“条件”，适用于 OR 子句优化。

WHERE 子句的所有条件都会被分析，以确定它们是否可以通过索引满足。要被索引使用，条件通常必须满足以下形式之一：

> ```sql
> *column* = *expression**column* IS *expression**column* > *expression**column* >= *expression**column* < *expression**column* <= *expression**expression* = *column**expression* IS *column**expression* > *column**expression* >= *column**expression* < *column**expression* <= *column**column* IN (*expression-list*) *column* IN (*subquery*) *column* IS NULL *column* LIKE *pattern**column* GLOB *pattern*
> ```

如果使用类似以下语句创建索引：

```sql
CREATE INDEX idx_ex1 ON ex1(a,b,c,d,e,...,y,z);

```

如果索引的初始列（例如列 a、b 等）出现在 WHERE 子句的条件中，则可能会使用该索引。索引的初始列必须与 **=** 或 **IN** 或 **IS** 操作符一起使用。使用的索引最右列可以包含不等式。对于使用的索引最右列，可以有多达两个不等式，这些不等式必须将该列的允许值夹在两个极端值之间。

并非必须在 WHERE 子句的每个条件中出现索引的每一列才能使用该索引。但是，索引使用的列之间不能有间隔。因此，对于上述示例的索引，如果没有约束列 c 的 WHERE 子句条件，则约束列 a 和 b 的条件可以与索引一起使用，但不能约束列 d 到 z 的条件。类似地，如果索引列位于仅通过不等式约束的列右侧，则通常不会使用索引列（用于索引目的）。（有关例外情况，请参见下面的 跳过扫描优化。）

在表达式索引的情况下，每当前文中出现"列"一词时，可以将其替换为"索引表达式"（即出现在 CREATE INDEX 语句中的表达式的副本），所有内容将依然有效。

## 2.1\. 索引术语使用示例

对于上述索引和类似的 WHERE 子句：

```sql
... WHERE a=5 AND b IN (1,2,3) AND c IS NULL AND d='hello'

```

索引的前四列 a、b、c 和 d 可以使用，因为这四列形成了索引的前缀，并且都受到等式约束的限制。

对于上述索引和类似的 WHERE 子句：

```sql
... WHERE a=5 AND b IN (1,2,3) AND c>12 AND d='hello'

```

只有索引的列 a、b 和 c 可以使用。列 d 无法使用，因为它位于 c 的右侧，并且 c 只受到不等式约束的限制。

对于上述索引和类似的 WHERE 子句：

```sql
... WHERE a=5 AND b IN (1,2,3) AND d='hello'

```

只有索引的列 a 和 b 可以使用。列 d 无法使用，因为列 c 没有约束，并且索引使用的列集中不应有空缺。

对于上述索引和类似的 WHERE 子句：

```sql
... WHERE b IN (1,2,3) AND c NOT NULL AND d='hello'

```

整个索引都无法使用，因为索引的最左列（列 "a"）没有约束。假设没有其他索引，上述查询将导致全表扫描。

对于上述索引和类似的 WHERE 子句：

```sql
... WHERE a=5 OR b IN (1,2,3) OR c NOT NULL OR d='hello'

```

由于 WHERE 子句中的术语由 OR 而不是 AND 连接，因此索引无法使用。此查询将导致全表扫描。但是，如果添加了另外三个包含列 b、c 和 d 作为最左列的索引，那么可能会应用 OR 子句优化。

# 3\. BETWEEN 优化

如果 WHERE 子句的一个术语具有以下形式：

> ```sql
> *expr1* BETWEEN *expr2* AND *expr3*
> ```

然后添加两个"虚拟"术语如下：

> ```sql
> *expr1* >= *expr2* AND *expr1* <= *expr3*
> ```

虚拟项仅用于分析，并且不会生成任何字节码。如果两个虚拟项最终都被用作索引的约束条件，那么原始的**BETWEEN**条件将被省略，并且对输入行不执行相应的测试。因此，如果**BETWEEN**条件最终被用作索引约束，那么该条件上的测试将永远不会执行。另一方面，虚拟项本身永远不会导致对输入行执行测试。因此，如果**BETWEEN**条件未被用作索引约束，而必须用于测试输入行时，*expr1*表达式只会被评估一次。

# 4\. OR 优化

通过 OR 而不是 AND 连接的 WHERE 子句约束可以以两种不同的方式处理。

## 4.1\. 将 OR 连接的约束转换为 IN 运算符

如果一个项由包含公共列名且由 OR 分隔的多个子项组成，如下所示：

> ```sql
> *column* = *expr1* OR *column* = *expr2* OR *column* = *expr3* OR ... 
> ```

然后该项被重写如下：

> ```sql
> *column* IN (*expr1*,*expr2*,*expr3*,...) 
> ```

然后重写的项可能继续使用常规的**IN**运算符规则来约束索引。请注意，*column*必须是每个 OR 连接子项中相同的列，尽管该列可以出现在**=**运算符的左侧或右侧。

## 4.2\. 单独评估 OR 约束并取其结果的并集

当且仅当先前描述的 OR 转换为 IN 运算符不起作用时，尝试第二个 OR 子句优化。假设 OR 子句由多个子项组成如下：

> ```sql
> *expr1* OR *expr2* OR *expr3*
> ```

个别子项可以是单个比较表达式，如**a=5**或**x>y**，也可以是 LIKE 或 BETWEEN 表达式，或者子项可以是 AND 连接的子子项的括号列表。每个子项都被分析，就好像它本身是整个 WHERE 子句，以便查看该子项是否可以单独建立索引。如果 OR 子句的每个子项都可以单独建立索引，那么 OR 子句可能被编码为使用单独的索引来评估 OR 子句的每个项。想象 SQLite 如何对每个 OR 子句项使用单独索引的一种方式是将 WHERE 子句重写如下：

> ```sql
>  rowid IN (SELECT rowid FROM *table* WHERE *expr1* UNION SELECT rowid FROM *table* WHERE *expr2* UNION SELECT rowid FROM *table* WHERE *expr3*) 
> ```

上述重写表达式是概念性的；包含 OR 的 WHERE 子句并不真的被这样重写。OR 子句的实际实现使用了一种更高效的机制，即使对于 WITHOUT ROWID 表或"rowid"不可访问的表也能工作。然而，上述语句捕捉到了实现的本质：使用单独的索引来查找每个 OR 子句项的候选结果行，最终结果是这些行的并集。

注意，在大多数情况下，SQLite 在查询的 FROM 子句中每个表只使用单个索引。这里描述的第二个 OR 子句优化是这一规则的例外。对于 OR 子句，每个子项可能会使用不同的索引。

对于任何给定的查询，这里描述的 OR 子句优化能够使用并不保证它会被使用。SQLite 使用基于成本的查询规划器，估算各种竞争查询计划的 CPU 和磁盘 I/O 成本，并选择它认为是最快的计划。如果 WHERE 子句中有许多 OR 项，或者一些 OR 子句子项的索引选择性不太高，则 SQLite 可能会决定更快地使用不同的查询算法，甚至是全表扫描。应用程序开发人员可以在语句上使用 EXPLAIN QUERY PLAN 前缀，以获取所选查询策略的高级概述。

# 5\. LIKE 优化

使用 LIKE 或 GLOB 运算符的 WHERE 子句项有时可以与索引一起使用，进行范围搜索，几乎就像 LIKE 或 GLOB 是 BETWEEN 运算符的替代品一样。关于这种优化有很多条件：

1.  LIKE 或 GLOB 的右侧必须是字符串文字或绑定到不以通配符字符开头的字符串文字的参数。

1.  使用数字值（而不是字符串或二进制大对象）来使 LIKE 或 GLOB 运算符为真是不可能的。这意味着：

    1.  LIKE 或 GLOB 运算符的左侧是具有 TEXT 亲和性的索引列的名称，或者

    1.  右侧的模式参数不能以减号（"-"）或数字开头。这个约束是由于数字在词典排序中不排序。例如：9<10 但'9'>'10'。

1.  用于实现 LIKE 和 GLOB 的内置函数不能使用 sqlite3_create_function() API 进行重载。

1.  对于 GLOB 运算符，必须使用内置的 BINARY 排序序列对列进行索引。

1.  对于 LIKE 操作符，如果启用了 case_sensitive_like 模式，则必须使用 BINARY 校对序列对列进行索引，或者如果禁用了 case_sensitive_like 模式，则必须使用内置的 NOCASE 校对序列进行索引。

1.  如果使用了 ESCAPE 选项，则 ESCAPE 字符必须是 ASCII 或 UTF-8 中的单字节字符。

LIKE 操作符有两种模式，可以通过 pragma 进行设置。默认模式是对 latin1 字符的大小写差异不敏感。因此，默认情况下，以下表达式为真：

```sql
'a' LIKE 'A'

```

如果 case_sensitive_like pragma 被启用如下：

```sql
PRAGMA case_sensitive_like=ON;

```

那么 LIKE 操作符会区分大小写，并且上述示例将评估为假。请注意，大小写不敏感仅适用于 latin1 字符 - 基本上是 ASCII 的 127 字节代码中的英语大写和小写字母。SQLite 中的国际字符集是区分大小写的，除非提供了应用定义的 校对序列 和 like() SQL 函数，以考虑非 ASCII 字符。如果提供了应用定义的校对序列和/或 like() SQL 函数，则此处描述的 LIKE 优化将永远不会生效。

LIKE 操作符默认情况下不区分大小写，因为这是 SQL 标准要求的。您可以通过编译时使用 SQLITE_CASE_SENSITIVE_LIKE 命令行选项来改变默认行为。

如果左侧命名的列使用内置的 BINARY 校对顺序进行索引，并且 case_sensitive_like 已打开，则可能发生 LIKE 优化。或者如果该列使用内置的 NOCASE 校对顺序进行索引，并且 case_sensitive_like 模式已关闭，则可能发生优化。这是仅有的两种情况，LIKE 运算符将在其下被优化。

GLOB 运算符始终区分大小写。GLOB 运算符左侧的列必须始终使用内置的 BINARY 校对顺序，否则不会尝试使用索引来优化该运算符。

仅当 GLOB 或 LIKE 运算符的右侧是文字字符串或已绑定到字符串文字的 参数 时，才会尝试 LIKE 优化。字符串文字不能以通配符开头；如果右侧以通配符字符开头，则不会尝试此优化。如果右侧是绑定到字符串的 参数，则仅在包含该表达式的 预处理语句 使用 sqlite3_prepare_v2() 或 sqlite3_prepare16_v2() 编译时才会尝试此优化。如果右侧是 参数，并且使用 sqlite3_prepare() 或 sqlite3_prepare16() 准备语句，则不会尝试 LIKE 优化。

假设在 LIKE 或 GLOB 操作符右侧的非通配符字符初始序列为*x*。我们使用单个字符来表示这个非通配符前缀，但读者应理解该前缀可以由多个字符组成。让*y*成为与/x/相同长度但大于*x*的最小字符串。例如，如果*x*是`'hello'`，那么*y*将是`'hellp'`。像这样的 LIKE 和 GLOB 优化包括添加两个虚拟术语：

> ```sql
> *column* >= *x* AND *column* < *y*
> ```

在大多数情况下，即使使用虚拟术语来约束索引，原始的 LIKE 或 GLOB 操作符仍然对每个输入行进行测试。这是因为我们不知道*x*前缀右侧可能施加的额外约束。但是，如果在*x*的右侧只有一个全局通配符，则会禁用原始的 LIKE 或 GLOB 测试。换句话说，如果模式如下所示：

> ```sql
> *column* LIKE *x*% *column* GLOB *x** 
> ```

那么当虚拟术语约束索引时，原始的 LIKE 或 GLOB 测试被禁用，因为在这种情况下，我们知道索引选中的所有行都将通过 LIKE 或 GLOB 测试。

注意，当 LIKE 或 GLOB 操作符右侧是参数，并且语句是使用 sqlite3_prepare_v2()或 sqlite3_prepare16_v2()准备的时，如果与上一次运行时的绑定到右侧参数发生了变化，语句将在每次运行的第一个 sqlite3_step()调用时自动重新解析和重新编译。这种重新解析和重新编译本质上与架构更改后发生的操作相同。重新编译是必要的，以便查询规划器可以检查绑定到 LIKE 或 GLOB 操作符右侧的新值，并确定是否应用上述优化。

# 6\. 跳跃扫描优化

一般规则是，只有在索引的最左列上有 WHERE 子句约束时，索引才是有用的。然而，在某些情况下，即使省略了索引的前几列于 WHERE 子句中，但后续列包含在内，SQLite 也能够使用该索引。

考虑一个如下所示的表：

```sql
CREATE TABLE people(
  name TEXT PRIMARY KEY,
  role TEXT NOT NULL,
  height INT NOT NULL, -- in cm
  CHECK( role IN ('student','teacher') )
);
CREATE INDEX people_idx1 ON people(role, height);

```

people 表中每个人在大型组织中都有一条记录。每个人通过 "role" 字段确定是 "student" 还是 "teacher"。表还记录了每个人的身高（单位：厘米）。角色和身高都被索引了。注意，索引的最左列不是非常选择性高 - 它只包含两个可能的值。

现在考虑一个查询，以找出组织中所有身高达到或超过 180cm 的人的姓名：

```sql
SELECT name FROM people WHERE height>=180;

```

由于查询的 WHERE 子句中没有索引的最左列，人们很容易得出结论认为此处无法使用索引。然而，SQLite 可以使用该索引。从概念上讲，SQLite 使用索引的方式类似于以下查询：

```sql
SELECT name FROM people
 WHERE role IN (SELECT DISTINCT role FROM people)
   AND height>=180;

```

或者这样：

```sql
SELECT name FROM people WHERE role='teacher' AND height>=180
UNION ALL
SELECT name FROM people WHERE role='student' AND height>=180;

```

上述的替代查询形式仅是概念性的。SQLite 实际上并没有转换查询。实际的查询计划如下：SQLite 定位到"role"的第一个可能的值，它可以通过倒回"people_idx1"索引到开头并读取第一条记录来实现。SQLite 将这个第一个"role"值存储在一个我们称之为"$role"的内部变量中。然后，SQLite 运行像这样的查询："SELECT name FROM people WHERE role=$role AND height>=180"。这个查询对索引的最左列有一个相等约束，因此可以使用索引解析该查询。一旦完成了该查询，SQLite 就使用"people_idx1"索引来定位"role"列的下一个值，使用的逻辑上类似于"SELECT role FROM people WHERE role>$role LIMIT 1"的代码。这个新的"role"值覆盖了$role 变量，这个过程重复，直到"role"的所有可能值都被检查完为止。

我们将这种索引使用方式称为"跳跃扫描"，因为数据库引擎基本上对索引进行全面扫描，但通过偶尔跳过到下一个候选值来优化扫描（使其不完全"全面"）。

如果 SQLite 知道索引的第一个或多个列包含许多重复值，它可能会在索引上使用跳跃扫描。如果索引的最左边列中重复值太少，那么简单地跳到下一个值，从而进行全表扫描，会比在索引上进行二分搜索定位到下一个左列值更快。

SQLite 只有在数据库上运行了 ANALYZE 命令后才能知道索引的左侧列中存在许多重复项。没有进行 ANALYZE 的结果，SQLite 必须猜测表中数据的“形状”，默认的猜测是索引的左侧列中每个值平均有 10 个重复项。只有在重复项约为 18 或更多时，跳跃扫描才会变得有利（即比完整表扫描更快）。因此，在未经分析的数据库上从不使用跳跃扫描。

# 7\. 连接

SQLite 将连接实现为嵌套循环。连接中嵌套循环的默认顺序是 FROM 子句中最左侧的表形成外层循环，最右侧的表形成内层循环。然而，如果改变循环顺序有助于选择更好的索引，SQLite 将会以不同的顺序嵌套循环。

内连接可以自由重新排序。然而外连接既不可交换也不可结合，因此不会重新排序。外连接左右两侧的内连接可能会被优化器重新排序，但外连接始终按照出现的顺序进行评估。

SQLite 特别对待 CROSS JOIN 运算符。理论上，CROSS JOIN 运算符是可交换的。然而，SQLite 选择永远不会重新排列 CROSS JOIN 中的表。这为程序员提供了一种机制，可以强制 SQLite 选择特定的循环嵌套顺序。

在选择连接中的表顺序时，SQLite 使用了一个高效的多项式时间算法图算法，详见下一代查询计划器文档。因此，SQLite 能够在微秒级别内规划具有 50 或 60 个表连接的查询。

连接重新排序是自动的，通常运行良好，程序员无需考虑它，特别是如果已经使用 ANALYZE 收集了关于可用索引的统计信息，尽管偶尔需要程序员提供一些提示。例如，考虑以下架构：

```sql
CREATE TABLE node(
   id INTEGER PRIMARY KEY,
   name TEXT
);
CREATE INDEX node_idx ON node(name);
CREATE TABLE edge(
   orig INTEGER REFERENCES node,
   dest INTEGER REFERENCES node,
   PRIMARY KEY(orig, dest)
);
CREATE INDEX edge_idx ON edge(dest,orig);

```

上述架构定义了一个有向图，可以在每个节点上存储名称。现在考虑针对此架构的查询：

```sql
SELECT *
  FROM edge AS e,
       node AS n1,
       node AS n2
 WHERE n1.name = 'alice'
   AND n2.name = 'bob'
   AND e.orig = n1.id
   AND e.dest = n2.id;

```

此查询请求的是从标记为"alice"的节点到标记为"bob"的节点的所有边的信息。SQLite 中的查询优化器基本上有两种实现此查询的选择。 （实际上有六种不同的选择，但我们在这里只考虑其中的两种。）下面的伪代码展示了这两种选择。

选项 1:

```sql
foreach n1 where n1.name='alice' do:
  foreach n2 where n2.name='bob' do:
    foreach e where e.orig=n1.id and e.dest=n2.id
      return n1.*, n2.*, e.*
    end
  end
end

```

选项 2:

```sql
foreach n1 where n1.name='alice' do:
  foreach e where e.orig=n1.id do:
    foreach n2 where n2.id=e.dest and n2.name='bob' do:
      return n1.*, n2.*, e.*
    end
  end
end

```

这两种实现选项中的每个循环都使用相同的索引来加快速度。这两个查询计划唯一的区别是循环嵌套的顺序。

那么哪个查询计划更好呢？结果表明答案取决于节点和边表中存在什么类型的数据。

让 alice 节点的数量为 M，bob 节点的数量为 N。考虑两种情况。在第一种情况下，M 和 N 都是 2，但每个节点上有数千条边。在这种情况下，更喜欢选项 1。使用选项 1，内循环检查节点对之间是否存在边，并在找到时输出结果。因为 alice 和 bob 节点只有 2 个，所以内循环只需运行四次，查询非常快。选项 2 在这里会花费更多时间。选项 2 的外部循环仅执行两次，但因为每个 alice 节点离开的边很多，中间循环必须迭代多次数千次。这将慢得多。因此在第一种情况下，我们更喜欢使用选项 1。

现在考虑 M 和 N 都是 3500 的情况。Alice 节点非常丰富。这时假设每个节点只通过一两条边连接。现在选项 2 更好。选择选项 2 时，外部循环仍然需要运行 3500 次，但中间循环每次仅需运行一两次，而内部循环每次可能根本不需要运行。因此，内部循环的总迭代次数约为 7000 次。另一方面，选项 1 需要分别运行外部循环和中间循环 3500 次，导致中间循环的迭代次数达到 1200 万次。因此，在第二种情况下，选项 2 几乎比选项 1 快了将近 2000 倍。

因此，您可以看到，根据表中数据的结构，查询计划 1 或查询计划 2 可能更好。SQLite 默认选择哪个查询计划呢？截至 3.6.18 版本，在未执行 ANALYZE 时，SQLite 将选择选项 2。如果运行 ANALYZE 命令以收集统计信息，则根据统计数据显示可能会选择不同的更快选项。

## 7.1\. 手动控制连接顺序

SQLite 几乎总是会自动选择最佳的连接顺序。开发者很少需要干预查询规划器以提供关于最佳连接顺序的提示。最佳策略是利用 PRAGMA optimize 确保查询规划器可以访问数据库中数据形状的最新统计信息。

本节描述了开发者可以控制 SQLite 中连接顺序的技术，以解决可能出现的任何性能问题。然而，除非迫不得已，不建议使用这些技术。

如果在运行 PRAGMA optimize 之后，SQLite 仍然选择了次优的连接顺序，请在[SQLite 社区论坛](https://sqlite.org/forum)上报告您的情况，以便 SQLite 的维护人员对查询优化器进行新的优化，从而避免需要手动干预。

### 7.1.1\. 使用 SQLITE_STAT 表手动控制查询计划

SQLite 允许高级程序员对优化器选择的查询计划进行控制。其中一种方法是在 sqlite_stat1 表中调整 ANALYZE 的结果。

### 7.1.2\. 使用 CROSS JOIN 手动控制查询计划

程序员可以通过使用 CROSS JOIN 操作符而非 JOIN、INNER JOIN、NATURAL JOIN 或逗号连接来强制 SQLite 使用特定的循环嵌套顺序。尽管 CROSS JOIN 在理论上是可交换的，但 SQLite 选择永远不会重新排序 CROSS JOIN 中的表。因此，CROSS JOIN 的左表始终相对于右表处于外部循环。

在以下查询中，优化器可以自由地重新排序 FROM 子句中的表：

```sql
SELECT *
  FROM node AS n1,
       edge AS e,
       node AS n2
 WHERE n1.name = 'alice'
   AND n2.name = 'bob'
   AND e.orig = n1.id
   AND e.dest = n2.id;

```

在同一个查询的逻辑等效形式中，将","替换为"CROSS JOIN"意味着表的顺序必须是 N1、E、N2。

```sql
SELECT *
  FROM node AS n1 CROSS JOIN
       edge AS e CROSS JOIN
       node AS n2
 WHERE n1.name = 'alice'
   AND n2.name = 'bob'
   AND e.orig = n1.id
   AND e.dest = n2.id;

```

在后一个查询中，查询计划必须是 option 2。请注意，您必须使用关键字"CROSS"以禁用表重排序优化；INNER JOIN、NATURAL JOIN、JOIN 和其他类似组合的行为与逗号连接相同，优化器可以根据需要重新排序表。（在外连接中也禁用了表重排序，但这是因为外连接不是可结合或可交换的。重新排序 OUTER JOIN 中的表会改变结果。）

参见 "The Fossil NGQP Upgrade Case Study"，这是另一个使用 CROSS JOIN 手动控制连接嵌套顺序的真实案例。文档后面的 query planner checklist 提供了关于手动控制查询规划器的进一步指导。

# 8\. 在多个索引之间进行选择

查询的 FROM 子句中的每个表最多可以使用一个索引（除非涉及到 OR-clause optimization）。SQLite 努力确保每个表至少使用一个索引。有时，同一表可能会有两个或更多个索引可供选择。例如：

```sql
CREATE TABLE ex2(x,y,z);
CREATE INDEX ex2i1 ON ex2(x);
CREATE INDEX ex2i2 ON ex2(y);
SELECT z FROM ex2 WHERE x=5 AND y=6;

```

对于上述的 SELECT 语句，优化器可以使用 ex2i1 索引来查找包含 x=5 的 ex2 表中的行，然后对每一行测试 y=6 条件。或者它可以使用 ex2i2 索引来查找包含 y=6 的 ex2 表中的行，然后对每一行测试 x=5 条件。

当面对两个或更多索引选择时，SQLite 会尝试估计使用每个选项执行查询所需的总工作量。然后选择估计工作量最少的选项。

为了帮助优化器更准确地估计使用各种索引所涉及的工作量，用户可以选择运行 ANALYZE 命令。ANALYZE 命令会扫描数据库中所有可能存在两个或多个索引选择的地方，并收集这些索引的选择性统计信息。此扫描收集的统计信息存储在名为 "**sqlite_stat**" 的特殊数据库表中。这些表的内容在数据库更改后不会更新，因此在进行重大更改后重新运行 ANALYZE 可能是明智的选择。ANALYZE 命令的结果仅对于在执行 ANALYZE 命令完成后打开的数据库连接可用。

各种 **sqlite_stat***N* 表包含了有关各种索引选择性的信息。例如，sqlite_stat1 表可能表明，对列 x 的等值约束平均将搜索空间减少到 10 行，而对列 y 的等值约束平均将搜索空间减少到 3 行。在这种情况下，SQLite 更倾向于使用 ex2i2 索引，因为该索引更具选择性。

## 8.1\. 使用一元 "+" 禁用 WHERE 子句条件

*注意：不推荐以这种方式禁用 WHERE 子句条件。这是一种解决方法。只有在迫不得已需要提高性能时才这样做。如果发现有必要使用这种解决方法的情况，请在 [SQLite Community Forum](https://sqlite.org/forum) 上报告情况，以便 SQLite 的维护者可以尽力改进查询规划器，以便在您的情况下不再需要这种解决方法。*

WHERE 子句的条件可以通过在列名前添加一元运算符 **+** 来手动禁用索引使用。这个一元 **+** 是一个空操作，在预编译语句中不会生成任何字节码。然而，这个一元 **+** 运算符将阻止条件限制索引。因此，在上面的例子中，如果查询被重写为：

```sql
SELECT z FROM ex2 WHERE +x=5 AND y=6;

```

对列 **x** 使用 **+** 运算符将阻止该条件限制索引。这将强制使用 ex2i2 索引。

请注意，一元 **+** 运算符也会从表达式中移除 类型亲和性，在某些情况下，这可能会导致表达式含义上的细微变化。在上面的例子中，如果列 **x** 具有 TEXT 亲和性，那么比较 "x=5" 将作为文本进行。**+** 运算符移除了亲和性。因此，比较 "**+x=5**" 将会将列 **x** 中的文本与数值 5 进行比较，并且始终为假。

## 8.2\. 范围查询

考虑一个略有不同的情况：

```sql
CREATE TABLE ex2(x,y,z);
CREATE INDEX ex2i1 ON ex2(x);
CREATE INDEX ex2i2 ON ex2(y);
SELECT z FROM ex2 WHERE x BETWEEN 1 AND 100 AND y BETWEEN 1 AND 100;

```

进一步假设列 x 包含的值分布在 0 到 1,000,000 之间，而列 y 包含的值则在 0 到 1,000 之间。在这种情况下，对列 x 的范围约束应该将搜索空间减少 10,000 倍，而对列 y 的范围约束仅应该将搜索空间减少 10 倍。因此，应优先选择 ex2i1 索引。

SQLite 将做出此决定，但前提是已使用 SQLITE_ENABLE_STAT3 或 SQLITE_ENABLE_STAT4 进行编译。SQLITE_ENABLE_STAT3 和 SQLITE_ENABLE_STAT4 选项导致 ANALYZE 命令在 sqlite_stat3 或 sqlite_stat4 表中收集列内容的直方图，并使用这些直方图更好地猜测用于上述范围约束的最佳查询。STAT3 和 STAT4 主要的区别在于，STAT3 仅记录索引的最左列的直方图数据，而 STAT4 则记录索引的所有列的直方图数据。对于单列索引，STAT3 和 STAT4 的工作方式相同。

如果约束条件的右侧是简单的编译时常量或 参数，而不是表达式，则直方图数据才有用。

直方图数据的另一个限制是，它仅适用于索引的最左列。考虑以下情况：

```sql
CREATE TABLE ex3(w,x,y,z);
CREATE INDEX ex3i1 ON ex2(w, x);
CREATE INDEX ex3i2 ON ex2(w, y);
SELECT z FROM ex3 WHERE w=5 AND x BETWEEN 1 AND 100 AND y BETWEEN 1 AND 100;

```

这里的不等式是在列 x 和 y 上，它们不是最左侧的索引列。因此，直方图数据收集的不是索引的最左列时，无助于帮助选择列 x 和 y 上的范围约束。

# 9\. 覆盖索引

在对行进行索引查找时，通常的过程是在索引上进行二分搜索以找到索引条目，然后从索引中提取 rowid，并使用该 rowid 在原始表上进行二分搜索。因此，典型的索引查找涉及两次二分搜索。然而，如果从表中提取的所有列在索引本身中已经可用，SQLite 将使用索引中包含的值，从而永远不会查找原始表行。这为每行节省了一次二分搜索，可以使许多查询运行速度加快一倍。

当一个索引包含查询所需的所有数据，并且原始表从不需要被查询时，我们称该索引为“覆盖索引”。

# 10\. ORDER BY 优化

SQLite 会尽可能使用索引来满足查询中的 ORDER BY 子句。当面临使用索引来满足 WHERE 子句约束或满足 ORDER BY 子句的选择时，SQLite 会进行与上述相同的成本分析，并选择它认为会导致最快答案的索引。

SQLite 还会尝试使用索引来帮助满足 GROUP BY 子句和 DISTINCT 关键字。如果连接的嵌套循环可以安排使得 GROUP BY 或 DISTINCT 中等效的行是连续的，那么 GROUP BY 或 DISTINCT 逻辑可以通过将当前行与前一行进行比较来确定当前行是否属于同一组或是否是不同的。这比将每行与所有先前行进行比较的替代方法要快得多。

## 10.1\. 通过索引实现部分 ORDER BY

如果一个查询包含带有多个条件的 ORDER BY 子句，可能会导致 SQLite 使用索引使得行按照 ORDER BY 中某些前缀的顺序输出，但是 ORDER BY 中的后续条件却无法满足。在这种情况下，SQLite 会进行阻塞排序。假设 ORDER BY 子句有四个条件，并且查询结果的自然顺序导致行按照前两个条件的顺序出现。当每行由查询引擎输出并进入排序器时，当前行中与 ORDER BY 前两个条件相对应的输出会与前一行进行比较。如果它们发生了变化，则当前排序完成并输出，并开始新的排序。这样可以实现稍快的排序。更大的优势是，需要在内存中保存的行数大大减少，从而降低了内存需求，并且在核心查询完成之前可以开始输出。

# 11\. 子查询展开

当子查询出现在 SELECT 的 FROM 子句中时，最简单的行为是将子查询评估为临时表，然后对临时表运行外部 SELECT。这样的计划可能是次优的，因为临时表将没有任何索引，外部查询（很可能是连接）将被迫在临时表上执行全表扫描，或者在临时表上构建一个 查询时索引，这两者都不太可能特别快。

为了解决这个问题，SQLite 试图展开 SELECT 的 FROM 子句中的子查询。这涉及将子查询的 FROM 子句插入到外部查询的 FROM 子句中，并重写外部查询中引用子查询结果集的表达式。例如：

```sql
SELECT t1.a, t2.b FROM t2, (SELECT x+y AS a FROM t1 WHERE z<100) WHERE a>5

```

使用查询展开进行重写如下：

```sql
SELECT t1.x+t1.y AS a, t2.b FROM t2, t1 WHERE z<100 AND a>5

```

必须满足一长串条件才能进行查询展开。一些约束通过斜体文本标记为过时。为了保留其他约束的编号，这些额外约束在文档中保留。

普通读者不必理解所有这些规则。这里的重点是展开规则是微妙且复杂的。多年来，由于过于激进的查询展开，已经出现了多个 bug。另一方面，复杂查询和/或涉及视图的查询在展开规则更为保守时性能往往会受到影响。

1.  *(过时)*

1.  *(过时)*

1.  如果子查询是 LEFT JOIN 的右操作数，则

    1.  子查询不能是联接，并且

    1.  子查询的 FROM 子句不能包含虚拟表，并且

    1.  外部查询不能是 DISTINCT。

1.  子查询不是 DISTINCT。

1.  *(过时 - 已被第 4 条约束取代)*

1.  *(过时)*

1.  子查询有一个 FROM 子句。

1.  子查询不使用 LIMIT 或外部查询不是联接。

1.  子查询不使用 LIMIT 或外部查询不使用聚合函数。

1.  *(过时)*

1.  子查询和外部查询不会同时具有 ORDER BY 子句。

1.  *(过时 - 已被第 3 条约束取代)*

1.  子查询和外部查询不会同时使用 LIMIT。

1.  子查询不使用 OFFSET。

1.  如果外部查询是复合选择的一部分，则子查询不能有 LIMIT 子句。

1.  如果外部查询是聚合查询，则子查询不能包含 ORDER BY。

1.  如果子查询是复合 SELECT，则

    1.  所有复合运算符必须是 UNION ALL，并且

    1.  子查询复合中没有术语可以是聚合函数或 DISTINCT，并且

    1.  子查询中的每个项必须有一个 FROM 子句，并且

    1.  外部查询不能是聚合查询或 DISTINCT 查询。

    1.  子查询不能包含窗口函数。

    1.  子查询不得位于 LEFT JOIN 的右侧。

    1.  如果子查询是外部查询的第一个元素，或者子查询的任何分支都不包含 RIGHT 或 FULL JOINs。

    1.  复合子查询的所有分支中对应的结果集表达式必须具有相同的 亲和性。父查询和子查询可能包含 WHERE 子句。在规则（11）、（12）和（13）的约束下，它们也可以包含 ORDER BY、LIMIT 和 OFFSET 子句。

1.  如果子查询是复合选择，则父查询的 ORDER BY 子句中的所有术语必须是对子查询列的简单引用。

1.  如果子查询使用 LIMIT，则外部查询可能不具有 WHERE 子句。

1.  如果子查询是复合选择，则不能使用 ORDER BY 子句。

1.  如果子查询使用 LIMIT，则外部查询可能不是 DISTINCT。

1.  子查询可能不是递归的公共表达式（CTE）。

1.  如果外部查询是递归的公共表达式（CTE），则子查询可能不是复合查询。

1.  *(已废弃)*

1.  子查询和外部查询都不能在结果集或 ORDER BY 子句中包含窗口函数。

1.  子查询可能不是 RIGHT 或 FULL OUTER JOIN 的右操作数。

1.  如果子查询不是父查询的第一个元素，则子查询可能不包含 FULL 或 RIGHT JOIN。有两个子情况：

    1.  子查询不是复合查询。

    1.  子查询是一个复合查询，并且 RIGHT JOIN 出现在复合查询的任何分支中。（另见（17g））。

1.  子查询不是材料化 CTE。

在视图用作每次使用视图时，查询展开是一个重要的优化，因为每次使用视图都被转换为一个子查询。

# 12\. 子查询协程

SQLite 实现 FROM 子句子查询的方式有三种：

1.  展平子查询到其外部查询中

1.  将子查询评估为一个存在于正在评估的 SQL 语句的持续时间内的临时表，然后对该临时表运行外部查询。

1.  在与外部查询并行运行的协程中评估子查询，根据需要向外部查询提供行。

本节描述第三种技术：将子查询实现为协程。

协程类似于子例程，因为它在与调用者相同的线程中运行，并最终将控制返回给调用者。不同之处在于，协程还具有在完成之前返回并在下次调用时恢复执行的能力。

当将子查询实现为协程时，生成字节码以将子查询实现为独立查询，但不是将结果行返回给应用程序，而是在计算每行后将控制返回给调用者。调用者可以将计算的一行作为其计算的一部分，然后在准备好下一行时再次调用协程。

协程优于将子查询的完整结果集存储在临时表中，因为协程使用的内存更少。使用协程时，只需要记住结果的单行，而对于临时表，必须存储所有结果行。此外，由于协程在外部查询开始工作之前不必运行到完成，因此输出的第一行可以更快地出现，如果整体查询在完成之前被放弃，那么总体工作量也会减少。

另一方面，如果必须多次扫描子查询的结果（例如，它只是连接中的一个表），那么最好使用临时表来记住子查询的整个结果，以避免多次计算子查询。

## 12.1\. 使用协程延迟工作直到排序后。

截至 SQLite 版本 3.21.0（2017-10-24），查询规划器将始终倾向于使用协程来实现包含 ORDER BY 子句的 FROM 子句子查询，当外部查询结果集是“复杂”的一部分时。此功能允许应用程序将昂贵的计算从排序器之前移至排序器之后，这可能会导致更快的操作。例如，考虑以下查询：

```sql
SELECT expensive_function(a) FROM tab ORDER BY date DESC LIMIT 5;

```

此查询的目标是计算表中最近五个条目的某些值。在上述查询中，“expensive_function()”在排序之前被调用，因此在表的每一行上都会被调用，即使最终由于 LIMIT 子句而被省略的行也是如此。可以使用协程来解决这个问题：

```sql
SELECT expensive_function(a) FROM (
  SELECT a FROM tab ORDER BY date DESC LIMIT 5
);

```

在修订后的查询中，由协程实现的子查询计算了“a”的最近五个值。这五个值从协程传递到外部查询，在那里只有应用程序关心的特定行上才调用了“expensive_function()”。

未来版本的 SQLite 查询规划器可能会变得足够智能，自动执行类似上述的转换，无论是从第一种形式到第二种形式，还是从第二种形式到第一种形式。截至 SQLite 版本 3.22.0（2018-01-22），如果外部查询结果集中不使用任何用户定义的函数或子查询，查询规划器将会展平子查询。但是，对于上述示例，SQLite 会按照查询的原样实现每个查询。

# 13\. 最小/最大值优化

包含单个 MIN()或 MAX()聚合函数，并且其参数是索引中最左侧列的查询可能只需执行单个索引查找，而不是扫描整个表。例如：

```sql
SELECT MIN(x) FROM table;
SELECT MAX(x)+1 FROM table;

```

# 14\. 自动查询时间索引

当没有索引可用来帮助查询的评估时，SQLite 可能会创建一个仅在单个 SQL 语句的持续时间内存在的自动索引。自动索引有时也称为“查询时间索引”。由于构建自动或查询时间索引的成本是 O(NlogN)（其中 N 是表中的条目数），而进行全表扫描的成本仅为 O(N)，因此只有在 SQLite 预计在 SQL 语句执行过程中查找超过 logN 次时才会创建自动索引。考虑以下例子：

```sql
CREATE TABLE t1(a,b);
CREATE TABLE t2(c,d);
-- Insert many rows into both t1 and t2
SELECT * FROM t1, t2 WHERE a=c;

```

在上面的查询中，如果 t1 和 t2 都大约有 N 行，那么在没有任何索引的情况下，查询将需要 O(N*N)的时间。另一方面，在表 t2 上创建一个索引需要 O(NlogN)的时间，并且使用该索引来评估查询需要额外的 O(NlogN)时间。在缺乏 ANALYZE 信息的情况下，SQLite 猜测 N 为一百万，因此认为构建自动索引将是更便宜的方法。

一个自动查询时间索引也可以用于子查询：

```sql
CREATE TABLE t1(a,b);
CREATE TABLE t2(c,d);
-- Insert many rows into both t1 and t2
SELECT a, (SELECT d FROM t2 WHERE c=b) FROM t1;

```

在这个例子中，t2 表用于子查询以翻译 t1.b 列的值。如果每个表包含 N 行，SQLite 预计子查询将运行 N 次，因此它认为首先在 t2 上构建一个自动的临时索引，然后使用该索引来满足子查询的 N 个实例会更快。

可以在运行时使用 automatic_index pragma 禁用自动索引功能。自动索引默认情况下是打开的，但可以通过 SQLITE_DEFAULT_AUTOMATIC_INDEX 编译时选项更改为默认关闭自动索引。可以通过使用 SQLITE_OMIT_AUTOMATIC_INDEX 编译时选项完全禁用创建自动索引的能力。

在 SQLite 版本 3.8.0（2013-08-26）及更高版本中，每次准备使用自动索引的语句时，都会向错误日志发送一个 SQLITE_WARNING_AUTOINDEX 消息。应用程序开发人员可以和应该利用这些警告来识别在架构中需要新持久索引的情况。

不要将自动索引与内部索引（其名称类似于“sqlite_autoindex_*table*_*N*”）混淆。有时会创建这些内部索引来实现主键约束或唯一约束。此处描述的自动索引仅存在于单个查询的持续时间内，从不持久化到磁盘，并且仅对单个数据库连接可见。内部索引是主键和唯一约束实现的一部分，具有长期持久性并持久化到磁盘，对所有数据库连接可见。术语“autoindex”出现在内部索引的名称中，出于传统原因，并不表示内部索引和自动索引有关联。

## 14.1\. 散列连接

自动索引几乎与[hash join](https://en.wikipedia.org/wiki/Hash_join)相同。唯一的区别是使用 B-Tree 而不是哈希表。如果你愿意说为自动索引构建的短暂 B-Tree 实际上只是一个花哨的哈希表，那么使用自动索引的查询就是一个哈希连接。

在这种情况下，SQLite 构建了一个临时索引，而不是哈希表，因为它已经具备了强大且高性能的 B-Tree 实现，而哈希表则需要额外添加。在低内存嵌入式设备上使用的库的大小会因为添加单独的哈希表实现而增加，而性能收益微乎其微。或许未来 SQLite 可能会通过增加哈希表实现来进行增强，但目前继续在客户端/服务器数据库引擎可能使用哈希连接的情况下使用自动索引似乎更好。

# 15\. WHERE 子句推送优化

如果子查询无法 展平化 到外部查询中，仍然可能通过将 WHERE 子句从外部查询推送到子查询中来增强性能。考虑一个例子：

```sql
CREATE TABLE t1(a INT, b INT);
CREATE TABLE t2(x INT, y INT);
CREATE VIEW v1(a,b) AS SELECT DISTINCT a, b FROM t1;

SELECT x, y, b
  FROM t2 JOIN v1 ON (x=a)
 WHERE b BETWEEN 10 AND 20;

```

视图 v1 由于是 DISTINCT，无法进行 展平化。必须将其作为子查询运行，并将结果存储在临时表中，然后在 t2 和临时表之间执行连接。推送优化将 "b BETWEEN 10 AND 20" 条件推送到视图中。这样可以使临时表变小，如果 t1.b 上有索引，则有助于子查询运行更快。结果的评估如下：

```sql
SELECT x, y, b
  FROM t2
  JOIN (SELECT DISTINCT a, b FROM t1 WHERE b BETWEEN 10 AND 20)
 WHERE b BETWEEN 10 AND 20;

```

WHERE 子句推送优化并非总是可用。例如，如果子查询包含 LIMIT，则从外部查询推送 WHERE 子句的任何部分可能会改变内部查询的结果。在实现这一优化的 pushDownWhereTerms() 程序中，源代码中的注释解释了其他限制。

不要将这种优化与 MySQL 中同名的优化混淆。MySQL 的推入优化改变了 WHERE 子句约束条件的评估顺序，使得那些可以仅使用索引而不必查找相应表行的约束条件首先评估，从而在约束条件失败时避免不必要的表行查找。为了消除歧义，SQLite 将其称为"MySQL 推入优化"。SQLite 确实也执行 MySQL 推入优化，除了 WHERE 子句推入优化之外。但本节的重点是 WHERE 子句推入优化。

# 16\. 外连接强度降低优化

外连接（LEFT JOIN、RIGHT JOIN 或 FULL JOIN）有时可以简化。LEFT JOIN 或 RIGHT JOIN 可以转换为普通（INNER）JOIN，或者 FULL JOIN 可能转换为 LEFT JOIN 或 RIGHT JOIN。这可以发生在 WHERE 子句中存在保证简化后得到相同结果的条件时。例如，如果 LEFT JOIN 右表中的任何列必须为非 NULL 才能使 WHERE 子句为真，则 LEFT JOIN 将被降级为普通 JOIN。

判断连接是否可以简化的定理证明器并不完美。有时会返回假负。换句话说，有时候它无法证明降低外连接强度是安全的，而实际上是安全的。例如，该证明器不知道如果其第一个参数为 NULL，则 datetime() SQL 函数始终返回 NULL，因此它不会意识到以下查询中的 LEFT JOIN 可以进行强度降低：

```sql
SELECT urls.url
  FROM urls
  LEFT JOIN
    (SELECT *
      FROM (SELECT url_id AS uid, max(retrieval_time) AS rtime
              FROM lookups GROUP BY 1 ORDER BY 1)
      WHERE uid IN (358341,358341,358341)
    ) recent
    ON u.source_seed_id = recent.xyz OR u.url_id = recent.xyz
 WHERE
     DATETIME(recent.rtime) > DATETIME('now', '-5 days');

```

有可能未来对证明器的增强可能使其能够识别出对某些内建函数的 NULL 输入始终导致 NULL 答案。然而，并非所有内建函数都具有该属性（例如 coalesce()），当然，证明器永远无法推理出应用程序定义的 SQL 函数。

# 17\. 省略 OUTER JOIN 优化

有时候，可以完全省略查询中的 LEFT JOIN 或 RIGHT JOIN 而不改变结果。如果以下条件全部为真，则可能会发生这种情况：

1.  查询不是一个聚合查询。

1.  要么查询是 DISTINCT，要么 OUTER JOIN 的 ON 或 USING 子句约束联接，以便仅匹配单行。

1.  在查询之外，LEFT JOIN 的右表或 RIGHT JOIN 的左表不在其自己的 USING 或 ON 子句之外的任何地方使用。

在视图中使用 OUTER JOIN 时经常出现 OUTER JOIN 消除的情况，然后视图被用于这样的方式，即左连接的右表或右连接的左表上的列没有被引用。

这里是省略 LEFT JOIN 的一个简单示例：

```sql
CREATE TABLE t1(ipk INTEGER PRIMARY KEY, v1);
CREATE TABLE t2(ipk INTEGER PRIMARY KEY, v2);
CREATE TABLE t3(ipk INTEGER PRIMARY KEY, v3);

SELECT v1, v3 FROM t1 
  LEFT JOIN t2 ON (t1.ipk=t2.ipk)
  LEFT JOIN t3 ON (t1.ipk=t3.ipk)

```

在上述查询中，表 t2 完全未使用，因此查询规划器能够实现查询，如同它是这样编写的：

```sql
SELECT v1, v3 FROM t1 
  LEFT JOIN t3 ON (t1.ipk=t3.ipk)

```

截至本文撰写时，仅消除了 LEFT JOIN。此优化尚未普遍应用于 RIGHT JOIN，因为 RIGHT JOIN 是 SQLite 相对较新的添加。这种不对称性可能会在未来的版本中得到纠正。

# 18\. 常量传播优化

当 WHERE 子句包含两个或多个由 AND 运算符连接的相等约束条件，且所有约束的亲和性均相同时，SQLite 可能利用等式的传递性构建新的 "虚拟" 约束，用于简化表达式和/或提高性能。这称为 "常量传播优化"。

例如，考虑以下架构和查询：

```sql
CREATE TABLE t1(a INTEGER PRIMARY KEY, b INT, c INT);
SELECT * FROM t1 WHERE a=b AND b=5;

```

SQLite 分析 "a=b" 和 "b=5" 约束，并推断如果这两个约束成立，则必然也成立 "a=5"。这意味着可以使用整数主键值为 5 快速查找所需的行。
