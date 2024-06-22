# 1\. Introduction

> 原文：[`sqlite.com/queryplanner-ng.html`](https://sqlite.com/queryplanner-ng.html)

"查询规划器（query planner）"的任务是找出执行 SQL 语句的最佳算法或"查询计划（query plan）"。从 SQLite 版本 3.8.0（2013-08-26）开始，查询规划器组件已重写，以提高运行速度并生成更好的计划。这次重写被称为"下一代查询规划器（NGQP）"。

本文概述了查询规划的重要性，描述了查询规划中固有的一些问题，并概述了 NGQP 如何解决这些问题。

NGQP 几乎总是优于传统的查询规划器。然而，可能存在一些传统应用程序不知不觉地依赖于传统查询规划器中未定义和/或次优行为，并且在这些传统应用程序上升级到 NGQP 可能会导致性能回退。这种风险已经被考虑，并提供了一个清单来减少风险并修复任何可能出现的问题。

本文重点介绍 NGQP。关于 SQLite 查询规划器的更一般概述，涵盖了 SQLite 的整个历史，请参阅"SQLite 查询优化器概述"和"索引工作原理"文档。

# 2\. 背景

对于简单查询针对单个表且只有少量索引的情况，通常有一个最佳算法的明显选择。但是对于更大更复杂的查询，如带有多个索引和子查询的多路连接，可能会有数百、数千或数百万种合理的计算结果的算法。查询规划器的任务是从这多种可能性中选择单一的"最佳"查询计划。

查询规划器是使得 SQL 数据库引擎如此令人惊奇有用和强大的关键。（这对所有 SQL 数据库引擎都是真实的，而不仅仅是 SQLite。）查询规划器使程序员免于选择特定查询计划的繁琐工作，从而使程序员能够将更多的精力集中在更高级的应用程序问题上，并为最终用户提供更多的价值。对于简单查询，在选择查询计划显而易见时，这是方便但并不是非常重要的。但随着应用程序、模式和查询变得越来越复杂，一个聪明的查询规划器可以极大地加速和简化应用程序开发的工作。能够告诉数据库引擎所需内容，并让数据库引擎找出检索该内容的最佳方式，这具有惊人的力量。

编写一个优秀的查询规划器更像是艺术而不是科学。查询规划器必须处理不完整的信息。它无法确定任何特定计划的执行时间，除非实际运行该计划。因此，在比较两个或多个计划以确定哪个“最佳”时，查询规划器必须做出一些猜测和假设，而这些猜测和假设有时会是错误的。一个优秀的查询规划器是那种经常能找到正确解决方案，以至于应用程序员很少需要介入的规划器。

## 2.1\. SQLite 中的查询规划

SQLite 使用嵌套循环计算连接，每个表在连接中使用一个循环。（在 WHERE 子句中的 IN 和 OR 运算符可能会插入额外的循环。SQLite 也会考虑这些情况，但出于简单起见，在本文中我们将忽略它们。）每个循环可能使用一个或多个索引来加速搜索，或者一个循环可能是一个“全表扫描”，读取表中的每一行。因此，查询规划分解为两个子任务：

1.  选择各种循环的嵌套顺序

1.  选择每个循环的良好索引

选择嵌套顺序通常是更具挑战性的问题。一旦确定了连接的嵌套顺序，为每个循环选择索引的选择通常是显而易见的。

## 2.2\. SQLite 查询规划器稳定性保证

当启用查询规划器稳定性保证（QPSG）时，SQLite 将始终针对任何给定的 SQL 语句选择相同的查询计划，条件是：

1.  数据库架构没有显著改变，如添加或删除索引，

1.  未重新运行 ANALYZE 命令，

1.  使用相同版本的 SQLite。

QPSG 默认情况下是禁用的。可以在编译时使用 SQLITE_ENABLE_QPSG 编译选项启用它，或者在运行时通过调用 sqlite3_db_config(db,SQLITE_DBCONFIG_ENABLE_QPSG,1,0) 来启用。

QPSG 意味着如果在测试期间所有查询都运行有效，并且应用程序未更改架构，那么 SQLite 不会突然决定开始使用不同的查询计划，可能导致应用发布后的性能问题。如果您的应用在实验室中运行良好，那么部署后它将继续以相同的方式工作。

客户端/服务器 SQL 数据库引擎通常不会做出这种保证。在客户端/服务器 SQL 数据库引擎中，服务器会跟踪表的大小和索引质量的统计信息，查询规划器利用这些统计信息来帮助选择最佳的执行计划。随着内容在数据库中的添加、删除或更改，这些统计数据将会演变，可能会导致查询规划器开始对某些特定查询使用不同的执行计划。通常情况下，新计划对数据结构的演变会更好。但有时新的查询计划会导致性能下降。在客户端/服务器数据库引擎中，通常会有数据库管理员（DBA）来处理这些罕见的问题。但是在像 SQLite 这样的嵌入式数据库中，没有 DBA 可以解决问题，因此 SQLite 努力确保部署后执行计划不会意外改变。

需要注意的是，更改 SQLite 版本可能会导致查询计划的变化。同一版本的 SQLite 将始终选择相同的查询计划，但是如果重新链接应用程序以使用不同版本的 SQLite，则查询计划可能会改变。在罕见情况下，SQLite 版本更改可能会导致性能回退。这是您应考虑静态链接应用程序与 SQLite 而不是使用可能在未经您知情或控制的情况下发生变化的系统范围内的 SQLite 共享库的一个原因。

另请参阅：

+   分析使用模式的推荐

+   PRAGMA 优化

# 3\. 一个困难案例

"TPC-H Q8" 是来自[事务处理性能委员会（Transaction Processing Performance Council）](http://www.tpc.org/tpch/)的一个测试查询。SQLite 版本 3.7.17 及更早版本的查询规划器并不为 TPC-H Q8 选择好的计划。经过判断，无论如何调整旧有的查询规划器，都无法解决这个问题。为了找到 TPC-H Q8 查询的好解决方案，并持续改进 SQLite 的查询规划器质量，有必要重新设计查询规划器。本节试图解释为何需要进行这一重新设计，以及 NGQP 如何不同并解决了 TPC-H Q8 问题。

## 3.1\. 查询详情

TPC-H Q8 是一个八路连接查询。正如前文所述，查询规划器的主要任务是找出这八个循环的最佳嵌套顺序，以尽量减少完成连接所需的工作量。对于 TPC-H Q8 的简化模型如下图所示：

<svg class="pikchr" viewBox="0 0 677.503 310.031"><text x="132" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">S</text> <text x="234" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">L</text> <text x="336" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">O</text> <text x="438" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">C</text> <text x="539" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">N1</text> <text x="641" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">R</text> <text x="234" y="205" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">P</text> <text x="30" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">N2</text> <text x="81" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">6.00</text> <text x="81" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">2.08</text> <text x="183" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">9.17</text> <text x="183" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">2.30</text> <text x="285" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">2.77</text> <text x="285" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">4.03</text> <text x="387" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">2.64</text> <text x="387" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">5.30</text> <text x="489" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">2.08</text> <text x="489" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">6.40</text> <text x="590" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">1.79</text> <text x="590" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">3.47</text> <text x="245" y="155" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">2.64</text> <text x="222" y="155" text-anchor="end" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">6.01</text> <text x="30" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="30" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 5.52</text> <text x="132" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="132" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 9.47</text> <text x="234" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="234" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 16.40</text> <text x="336" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="336" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 13.87</text> <text x="438" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="438" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 12.56</text> <text x="539" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="539" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 5.52</text> <text x="641" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="641" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 3.56</text> <text x="234" y="293" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="234" y="256" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 7.71</text></svg>

在图表中，查询的 FROM 子句中的每个 8 个表都用一个大圆圈表示，标有 FROM 子句术语的标签：N2、S、L、P、O、C、N1 和 R。图中的弧线代表了假设弧线起点在外部循环时计算每个术语的估计成本。例如，将 S 循环作为 L 的内部循环的成本为 2.30，而将 S 循环作为 L 的外部循环的成本为 9.17。

这里的“成本”是对数的。对于嵌套循环，工作是相乘而不是相加。但通常将图形视为具有加法权重，因此图表显示各种成本的对数。图表显示 S 位于 L 内部的成本优势约为 6.87，但这意味着当 S 循环在 L 循环内部而不是外部时，查询运行大约快 963 倍。

从带有"*"标签的小圆圈指向的箭头表示在没有依赖关系的情况下运行每个循环的成本。最外层循环必须使用此*-成本。内部循环可以选择使用*-成本或者假设另一个术语是外部循环的成本，以给出最佳结果。可以将*-成本看作是一种简化表示，表示图中每个其他节点都有一条弧线。因此，该图是“完全的”，意味着图中的每对节点之间都有弧线（一些显式的，一些隐含的）。

寻找最佳查询计划的问题相当于找到通过图中每个节点恰好一次的最小成本路径。

（顺便说一句：上述 TPC-H Q8 图中的成本估算是由 SQLite 3.7.16 中的查询规划器计算的，并使用自然对数进行转换。）

## 3.2\. 复杂性

上面所述的查询规划器问题的展示是简化的。成本是估算的。在实际运行循环之前，我们无法知道运行循环的真实成本。SQLite 根据 WHERE 子句中找到的索引和约束的可用性对运行循环的成本进行猜测。这些猜测通常相当准确，但有时可能会偏离。使用 ANALYZE 命令收集关于数据库的额外统计信息，有时可以使 SQLite 更好地猜测成本。

成本由多个数字组成，而不是如图所示的单个数字。SQLite 为每个在不同时间应用的循环计算几种不同的估计成本。例如，一旦查询开始，就会产生“设置”成本。设置成本是为表计算查询时间索引（查询时间索引）而发生的成本，如果表尚无索引。然后是运行每个循环步骤的成本。最后，还有循环生成的行数的估计，这是在估算内部循环成本时需要的信息。如果查询有 ORDER BY 子句，则可能涉及排序成本。

在一般查询中，依赖关系不需要在单个循环上，因此依赖关系的矩阵可能无法表示为图形。例如，WHERE 子句中的一个约束可能是 S.a=L.b+P.c，这意味着 S 循环必须是 L 和 P 的内部循环。这种依赖关系无法绘制为图形，因为一个弧线无法同时起源于两个或更多的节点。

如果查询包含 ORDER BY 子句或 GROUP BY 子句，或者查询使用 DISTINCT 关键字，则选择通过图形的路径使行自然按排序顺序出现将是有利的，这样就不需要单独的排序步骤。自动消除 ORDER BY 子句可以产生很大的性能差异，因此这是需要在完整实现中考虑的另一个因素。

在 TPC-H Q8 查询中，设置成本都可以忽略不计，所有依赖都在各个单独节点之间，且没有 ORDER BY、GROUP BY 或 DISTINCT 子句。因此对于 TPC-H Q8，上述图形是需要计算的合理表示。一般情况涉及大量额外的复杂性，为了清晰起见，在本文的其余部分将忽略这些。

## 3.3\. 寻找最佳查询计划

在 版本 3.8.0（2013-08-26）之前，SQLite 在寻找最佳查询计划时总是使用“最近邻”或“NN”启发式方法。NN 启发式方法在遍历图形时总是选择成本最低的弧作为下一步。NN 启发式方法在大多数情况下表现出色。而且 NN 很快，因此 SQLite 能够快速找到即使是大型 64 路连接的良好计划。相比之下，其他执行更广泛搜索的 SQL 数据库引擎在连接表数超过 10 或 15 时往往会陷入困境。

不幸的是，NN 为 TPC-H Q8 计算的查询计划并不是最优的。使用 NN 计算的计划是 R-N1-N2-S-C-O-L-P，成本为 36.92。前面句子中的符号意味着 R 表在外部循环中运行，N1 在下一个内部循环中，N2 在第三个循环中，依此类推，最内层循环是 P。通过详尽搜索找到的图中最短路径是 P-L-O-C-N1-R-S-N2，成本为 27.38。差异看起来可能不大，但请记住成本是对数级别的，因此最短路径几乎比使用 NN 启发式方法找到的路径快 750 倍。

这个问题的一个解决方案是改变 SQLite，使其对最佳路径进行详尽搜索。但详尽搜索需要时间与 K! 成正比（其中 K 是连接中的表数），所以当连接表数超过 10 时，运行 sqlite3_prepare() 的时间变得非常长。

## 3.4\. N 最近邻居或 "N3" 启发式方法

NGQP 使用了一个新的启发式方法来寻找图中的最佳路径："N 最近邻居"（以下简称 "N3"）。使用 N3，算法在每一步都跟踪当前步骤中的 N 条最佳路径，其中 N 是一个小整数。

假设 N=4。对于 TPC-H Q8 图，第一步是找到访问图中任何单个节点的四条最短路径：

> R (cost: 3.56)
> 
> N1 (cost: 5.52)
> 
> N2 (cost: 5.52)
> 
> P (cost: 7.71)

第二步是找到访问两个节点的四条最短路径，以前一步中的四条路径之一开始。如果两条或更多路径是等效的（它们访问的节点集合相同，尽管可能顺序不同），则仅保留第一个和成本最低的路径。我们有：

> R-N1 (cost: 7.03)
> 
> R-N2 (cost: 9.08)
> 
> N2-N1 (cost: 11.04)
> 
> R-P {cost: 11.27}

第三步从四条最短的两节点路径开始，并找到四条最短的三节点路径：

> R-N1-N2 (cost: 12.55)
> 
> R-N1-C (cost: 13.43)
> 
> R-N1-P (成本：14.74)
> 
> R-N2-S (成本：15.08)

等等。在 TPC-H Q8 查询中有 8 个节点，因此这个过程总共重复 8 次。在 K 路连接的一般情况下，存储需求是 O(N)，计算时间是 O(K*N)，比 O(2^K)的精确解法快得多。

但是选择多少的 N 值呢？可以尝试 N=K。这使得算法为 O(K²)，实际上仍然相当高效，因为 K 的最大值为 64，而 K 很少超过 10。但这对于 TPC-H Q8 问题来说还不够。在 TPC-H Q8 上，当 N=8 时，N3 算法找到了成本为 29.78 的解决方案 R-N1-C-O-L-S-N2-P。这比 NN 有了很大的改进，但仍不是最优解。当 N 为 10 或更大时，N3 找到了 TPC-H Q8 的最优解。

NGQP 的初始实现选择简单查询时的 N=1，两表连接时的 N=5，三个或更多表连接时的 N=10。这种选择 N 的公式可能会在后续版本中更改。

# 4\. 升级到 NGQP 的风险

*→ 更新：**此部分已过时，仅供历史参考。** 当 NGQP 刚推出时，此部分非常重要。但是十年过去了，NGQP 已成功部署到数十亿设备上，并且每个人都已经升级，且未曾报告任何性能退化问题给 SQLite 开发者。升级的风险已经消失。此部分仅作历史参考。现代读者可以**跳转至查询规划器检查清单**。←*

对于大多数应用程序，从传统查询规划器升级到 NGQP 几乎不需要思考或努力。只需用新版本的 SQLite 替换旧版本并重新编译，应用程序将运行得更快。无需更改 API，也无需修改编译程序。

但是，像任何查询规划器的更改一样，升级到 NGQP 也会带来引入性能退化的小风险。问题并非 NGQP 不正确、有 bug 或比传统查询规划器差。只要有关索引选择性的可靠信息，NGQP 应该总是能选择与以前一样好或更好的计划。问题在于，一些应用可能使用质量低下且选择性低的索引，而没有运行 ANALYZE。旧的查询规划器每个查询考虑的实现较少，因此它们可能通过愚蠢的运气找到了一个好的计划。另一方面，NGQP 考虑了更多的查询计划可能性，它可能会选择在理论上更好的不同查询计划，假设有良好的索引，但由于数据的形状，实际上会导致性能退化。

关键点：

+   与先前的查询规划器相比，只要 NGQP 能够访问 ANALYZE 数据并且准确，它总是能找到一个相等或更好的查询计划，该数据存储在 SQLITE_STAT1 文件中。

+   NGQP 只要模式不包含在索引的最左列上有超过大约 10 或 20 行相同值的索引，总是能找到一个良好的查询计划。

并非所有应用程序都满足这些条件。幸运的是，即使没有这些条件，NGQP 通常仍能找到良好的查询计划。然而，偶尔会出现（很少）性能退化的情况。

## 4.1\. 案例研究：将 Fossil 升级至 NGQP

[Fossil DVCS](http://www.fossil-scm.org/)是用于跟踪所有 SQLite 源代码的版本控制系统。Fossil 存储库是一个 SQLite 数据库文件。（读者被邀请思考这种递归作为一个独立的练习。）Fossil 既是 SQLite 的版本控制系统，也是 SQLite 的测试平台。每当对 SQLite 进行增强时，Fossil 是首批应用程序之一来测试和评估这些增强。因此，Fossil 是 NGQP 的早期采用者之一。

不幸的是，NGQP 在 Fossil 中引发了性能回归。

Fossil 提供的众多报告之一是显示单个分支更改时间线的报告，显示该分支的所有合并情况。请参见[`www.sqlite.org/src/timeline?nd&n=200&r=trunk`](https://www.sqlite.org/src/timeline?nd&n=200&r=trunk)以查看此类报告的典型示例。生成此类报告通常只需几毫秒。但在升级到 NGQP 后，我们注意到这一报告对存储库的 trunk 分支的处理时间接近 10 秒。

生成分支时间线的核心查询如下所示。（读者无需理解此查询的详细内容。评论将随后提供。）

> ```sql
> SELECT
>      blob.rid AS blobRid,
>      uuid AS uuid,
>      datetime(event.mtime,'localtime') AS timestamp,
>      coalesce(ecomment, comment) AS comment,
>      coalesce(euser, user) AS user,
>      blob.rid IN leaf AS leaf,
>      bgcolor AS bgColor,
>      event.type AS eventType,
>      (SELECT group_concat(substr(tagname,5), ', ')
>         FROM tag, tagxref
>        WHERE tagname GLOB 'sym-*'
>          AND tag.tagid=tagxref.tagid
>          AND tagxref.rid=blob.rid
>          AND tagxref.tagtype>0) AS tags,
>      tagid AS tagid,
>      brief AS brief,
>      event.mtime AS mtime
>   FROM event CROSS JOIN blob
>  WHERE blob.rid=event.objid
>    AND (EXISTS(SELECT 1 FROM tagxref
>                 WHERE tagid=11 AND tagtype>0 AND rid=blob.rid)
>         OR EXISTS(SELECT 1 FROM plink JOIN tagxref ON rid=cid
>                    WHERE tagid=11 AND tagtype>0 AND pid=blob.rid)
>         OR EXISTS(SELECT 1 FROM plink JOIN tagxref ON rid=pid
>                    WHERE tagid=11 AND tagtype>0 AND cid=blob.rid))
>  ORDER BY event.mtime DESC
>  LIMIT 200;
> 
> ```

此查询并不特别复杂，但即使如此，它替换了数百甚至可能数千行的过程性代码。查询的要点是：扫描 EVENT 表，寻找满足以下三个条件之一的最近的 200 次检入：

1.  检入项具有一个“trunk”标签。

1.  检入项具有一个带有“trunk”标签的子项。

1.  检入项具有一个具有“trunk”标签的父项。

第一个条件导致所有主干检查点被显示，第二和第三个条件导致与主干合并或分叉的检查点也被包含在内。这三个条件由查询的 WHERE 子句中的三个 OR 连接的 EXISTS 语句实现。NGQP 导致的减速是由第二和第三个条件引起的。问题在每个条件中都是相同的，因此我们只会检查第二个条件。第二个条件的子查询可以重写（进行轻微和无实质性简化）如下：

> ```sql
> SELECT 1
>   FROM plink JOIN tagxref ON tagxref.rid=plink.cid
>  WHERE tagxref.tagid=$trunk
>    AND plink.pid=$ckid;
> 
> ```

PLINK 表保存了检查点之间的父子关系。TAGXREF 表将标签映射到检查点。为了参考，这两个表的相关部分模式如下所示：

> ```sql
> CREATE TABLE plink(
>   pid INTEGER REFERENCES blob,
>   cid INTEGER REFERENCES blob
> );
> CREATE UNIQUE INDEX plink_i1 ON plink(pid,cid);
> 
> CREATE TABLE tagxref(
>   tagid INTEGER REFERENCES tag,
>   mtime TIMESTAMP,
>   rid INTEGER REFERENCE blob,
>   UNIQUE(rid, tagid)
> );
> CREATE INDEX tagxref_i1 ON tagxref(tagid, mtime);
> 
> ```

只有两种合理的实现此查询的方式。（还有许多其他可能的算法，但其他算法都不适合成为“最佳”算法的候选者。）

1.  找到所有 $ckid 的子节点，并测试每一个是否具有 $trunk 标签。

1.  找到所有具有 $trunk 标签的检查点，并测试每一个是否是 $ckid 的子节点。

直觉上，我们人类理解算法-1 是最好的。每个检查点可能只有几个子项（一个子项是最常见的情况），每个子项可以在对数时间内检测“trunk”标签。事实上，在实践中，算法-1 是更快的选择。但是 NGQP 没有直觉。NGQP 必须使用严格的数学，而数学上算法-2 稍微更好。这是因为在没有其他信息的情况下，NGQP 必须假设索引 PLINK_I1 和 TAGXREF_I1 的质量相等且选择性相等。算法-2 使用 TAGXREF_I1 索引的一个字段和 PLINK_I1 索引的两个字段，而算法-1 只使用每个索引的第一个字段。由于算法-2 使用更多的索引材料，NGQP 正确判断它是更好的算法。分数非常接近，算法-2 略微领先于算法-1。但是在这里，算法-2 确实是正确的选择。

不幸的是，算法-2 在这个应用中比算法-1 慢。

索引质量不均等是问题所在。检查点通常只有一个子项。因此，PLINK_I1 的第一个字段通常只能将搜索范围缩小到单行。但是标记为“trunk”的检查点有成千上万个，因此 TAGXREF_I1 的第一个字段在缩小搜索范围方面帮助不大。

除非在数据库上运行了 ANALYZE，否则 NGQP 无法知道 TAGXREF_I1 在此查询中几乎没有用处。ANALYZE 命令会收集有关各种索引质量的统计信息，并将这些统计信息存储在 SQLITE_STAT1 表中。有了这些统计信息，NGQP 很容易选择算法-1 作为最佳算法，优势很大。

为什么传统的查询规划器不选择算法-2？很简单：因为 NN 算法甚至没有考虑过算法-2。规划问题的图表如下：

<svg class="pikchr" viewBox="0 0 523.232 186.76"><text x="55" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">P</text> <text x="157" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">T</text> <text x="106" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">4.8</text> <text x="106" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">4.4</text> <text x="55" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="55" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 4.9</text> <text x="157" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="157" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 5.2</text> <text x="106" y="171" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="125%" dominant-baseline="central">without ANALYZE</text> <text x="384" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">P</text> <text x="486" y="104" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="156.25%" dominant-baseline="central">T</text> <text x="435" y="84" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">4.4</text> <text x="435" y="123" text-anchor="middle" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central">3.8</text> <text x="384" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="384" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 3.9</text> <text x="486" y="16" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="486" y="53" text-anchor="start" fill="rgb(0,0,0)" font-size="80%" dominant-baseline="central"> 6.1</text> <text x="435" y="171" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" font-size="125%" dominant-baseline="central">with ANALYZE</text></svg>

在左侧的“without ANALYZE”情况下，NN 算法选择将循环 P（PLINK）作为外部循环，因为 4.9 小于 5.2，导致路径 P-T，即算法-1。NN 每次只查看每一步的单一最佳选择，因此完全忽略了 5.2 + 4.4 比 4.9 + 4.8 稍便宜的事实。但是 N3 算法会为双向连接保留最佳的 5 条路径，因此最终选择路径 T-P，因为其总体成本稍低。路径 T-P 是算法-2。

请注意，使用 ANALYZE 后，成本估算与实际更好地对齐，并且 NN 和 N3 都选择算法-1。

（附注：最近两个图表中的成本估算是由 NGQP 使用基于 2 的对数和与传统查询规划器稍有不同的成本假设计算的。因此，这两个图表中的成本估算与 TPC-H Q8 图表中的成本估算并不直接可比。）

## 4.2\. 修复问题

运行 ANALYZE 对仓库数据库立即解决了性能问题。然而，我们希望 Fossil 能够健壮并且始终快速工作，无论其仓库是否已分析。因此，查询被修改为使用 CROSS JOIN 操作符而不是普通的 JOIN 操作符。SQLite 不会重新排序 CROSS JOIN 的表格。这是 SQLite 的长期特性，专门设计用于让有经验的程序员能够强制执行特定的循环嵌套顺序。一旦将连接改为 CROSS JOIN（只增加了一个关键词），NGQP 就会被迫选择更快的算法-1，无论是否使用 ANALYZE 收集了统计信息。

我们说算法-1 更"快"，但这并不完全正确。在通常的存储库中，算法-1 更快，但也可以构建一个存储库，其中每次检入位于不同且唯一命名的分支，并且所有检入都是根检入的子检入。在这种情况下，TAGXREF_I1 会比 PLINK_I1 更具选择性，算法-2 确实是更快的选择。但是这样的存储库在实践中很不可能出现，因此在这种情况下，使用 CROSS JOIN 语法进行硬编码循环嵌套顺序是解决问题的合理方法。

## 4.3\. 更新 2017: 更好的修复方案

前文撰写于 2013 年初，早于 SQLite 版本 3.8.0 的首次发布。这段文字添加于 2021 年中。虽然之前的讨论仍然有效，但查询规划器已经进行了大量改进，使得整个章节大多已经不再重要。

2017 年，Fossil 进行了增强，以利用新的 PRAGMA optimize 语句。每当 Fossil 准备关闭与其存储库的数据库连接时，它首先运行"PRAGMA optimize"，这将导致如有必要则运行 ANALYZE。通常情况下不需要运行 ANALYZE，因此这样做不会带来可测量的性能损失。但是偶尔可能会在存储库数据库的几个表上运行 ANALYZE。因此，Fossil 中不再出现类似于这里描述的查询规划问题。定期运行 ANALYZE 以保持 sqlite_stat1 表的更新意味着不再需要手动调整查询。我们很久以前就不再在 Fossil 中调整查询了。

因此，目前建议避免类似问题的方法是在每次关闭数据库连接前简单地运行"PRAGMA optimize"。或者，如果您的应用程序长时间运行且不关闭任何数据库连接，则每天运行一次"PRAGMA optimize"。还要考虑在任何模式更改后运行"PRAGMA optimize"。

# 5\. 避免或修复查询规划器问题的检查列表

1.  **不要惊慌！** 查询规划器选择较差计划的情况实际上非常少见。您不太可能在应用程序中遇到任何问题。如果没有性能问题，您不需要担心这些问题。

1.  **创建适当的索引。** 大多数 SQL 性能问题不是由于查询规划器的问题，而是由于缺乏适当的索引引起的。确保索引能够帮助所有大查询。大多数性能问题可以通过一两个 CREATE INDEX 命令解决，而无需更改应用程序代码。

1.  **避免创建低质量的索引。** 低质量的索引（根据此检查列表的定义）是指表中左列有超过 10 或 20 行具有相同值的索引。特别是，避免将布尔或"enum"列用作索引的左列。

    在本文档前面部分描述的 Fossil 性能问题是由于 TAGXREF 表中 TAGXREF_I1 索引的左列（TAGID 列）具有超过一万个相同值的条目引起的。

1.  **如果必须使用低质量的索引，请确保运行 ANALYZE。** 只要查询规划器知道索引质量低，低质量的索引就不会让查询规划器混淆。查询规划器知道这一点的方式是通过 SQLITE_STAT1 表的内容，该内容由 ANALYZE 命令计算得出。

    当然，只有在数据库中有大量内容的情况下，**ANALYZE** 才能有效地发挥作用。当创建一个新的数据库，并且预计会积累大量数据时，可以运行命令"ANALYZE sqlite_schema"来创建**SQLITE_STAT1**表，然后通过普通的 INSERT 语句预先填充 sqlite_stat1 表，这些内容描述了你的应用程序的典型数据库，可能是在实验室中对一个已经填充良好的模板数据库运行 ANALYZE 后提取的内容。或者，你也可以在关闭数据库连接之前运行"PRAGMA optimize"，这样 ANALYZE 会根据需要自动运行，以保持 sqlite_stat1 表的最新状态。

1.  -   **为你的代码添加仪器。** 添加逻辑，可以让你快速简单地了解哪些查询花费了太多时间。然后专注于那些具体的查询。

    * * *

    *更新 2024：查询规划器经过多年的改进，你几乎不需要使用以下描述的任何技巧。虽然以下功能仍然可用，用于向后兼容，但你不应该使用它们。如果确实发现查询计划次优，敬请在[SQLite Forum](https://sqlite.org/forum)上向 SQLite 开发者报告，以便他们尝试解决问题。换句话说：***从这里停止阅读！

    为了帮助你停止阅读，这个清单的其余部分现在已经变灰。

    * * *

1.  使用 unlikely() 和 likelihood() SQL 函数。SQLite 通常假定 WHERE 子句中不能使用索引的条款有很强的真实概率。如果此假设是不正确的，则可能导致子优化的查询计划。unlikely() 和 likelihood() SQL 函数可用于向查询规划器提供关于 WHERE 子句中可能不真实的条款的提示，从而帮助查询规划器选择最佳的计划。

1.  使用 CROSS JOIN 语法来强制在可能使用未分析数据库中的低质量索引的查询上执行特定的循环嵌套顺序。SQLite 特别对待 CROSS JOIN 操作符，强制左侧的表相对于右侧的表成为外部循环。

1.  使用一元“+”运算符来排除 WHERE 子句中的条件。如果查询规划器坚持为特定查询选择质量较差的索引，而另一个质量更高的索引可用，则在 WHERE 子句中谨慎使用一元“+”运算符可以迫使查询规划器避开质量较差的索引。尽量避免使用这种技巧，特别是在应用程序开发周期的早期阶段要格外小心。请注意，如果类型亲和性涉及，向等式表达式添加一元“+”运算符可能会改变该表达式的结果。

1.  使用 INDEXED BY 语法来强制在问题查询中选择特定的索引。与前两点类似，尽量避免此步骤，特别是在开发早期，因为这显然是一种过早的优化。

# 6\. 摘要

在 SQLite 中，查询规划器通常能够很好地选择快速算法来运行您的 SQL 语句。这对于传统的查询规划器来说是正确的，甚至对于新的 NGQP 来说更为如此。偶尔会出现一种情况，即由于信息不完整，查询规划器选择了一个次优的执行计划。相比传统的查询规划器，使用 NGQP 出现这种情况的频率会更少，但仍然可能发生。只有在这些罕见情况下，应用程序开发人员才需要介入，帮助查询规划器做正确的事情。在普通情况下，NGQP 只是 SQLite 的一项新增功能，使应用程序运行速度稍快，而不需要开发人员进行新的思考或操作。
