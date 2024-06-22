# 1\. 概述

> 原文：[`sqlite.com/rtree.html`](https://sqlite.com/rtree.html)

[R-Tree](http://en.wikipedia.org/wiki/R-tree) 是一种专门设计用于执行范围查询的特殊索引。R-Trees 最常用于地理空间系统，其中每个条目都是具有最小和最大 X 和 Y 坐标的矩形。给定一个查询矩形，R-Tree 能够快速找到所有包含在查询矩形内或与查询矩形重叠的条目。这个想法可以很容易地扩展到三维，用于 CAD 系统。R-Trees 也用于时间域范围查找。例如，假设一个数据库记录了大量事件的开始和结束时间。R-Tree 能够快速找到在给定时间间隔内任何时间处于活动状态的所有事件，或者在特定时间间隔内开始的所有事件，或者在给定时间间隔内既开始又结束的所有事件，依此类推。

R-Tree 的概念源于[Toni Guttman](https://link.springer.com/referenceworkentry/10.1007/978-0-387-35973-1_1151)：*R-Trees: A Dynamic Index Structure for Spatial Searching*，1984 年 ACM SIGMOD 国际数据管理会议，第 47-57 页。在 SQLite 中找到的实现是 Guttman 原始想法的一个改进，通常称为“R*Trees”，由 Norbert Beckmann, Hans-Peter Kriegel, Ralf Schneider, Bernhard Seeger 描述：*The R*-Tree: An Efficient and Robust Access Method for Points and Rectangles.* SIGMOD Conference 1990: 322-331.

# 2\. 编译 R*Tree 模块

SQLite R*Tree 模块的源代码包含在 amalgamation 中。但是，具体取决于配置选项和您使用的 SQLite 版本，它可能已启用或未启用。要确保 R*Tree 模块已启用，只需在编译时定义 SQLITE_ENABLE_RTREE C 预处理宏。使用许多编译器，可以通过将选项“-DSQLITE_ENABLE_RTREE=1”添加到编译器命令行来实现此目的。

# 3\. 使用 R*Tree 模块

SQLite 的 R*Tree 模块被实现为一个 virtual table。每个 R*Tree 索引都是一个虚拟表，具有 3 到 11 个奇数列。第一列始终是一个 64 位有符号整数主键。其他列是成对出现的，每个维度包含该维度的最小和最大值。因此，一个一维的 R*Tree 有 3 列，一个二维的有 5 列，一个三维的有 7 列，一个四维的有 9 列，一个五维的有 11 列。SQLite 的 R*Tree 实现不支持超过 5 维的 R*Trees。

一个 SQLite R*Tree 的第一列类似于普通 SQLite 表的整数主键列。它只能存储 64 位有符号整数值。向这一列插入 NULL 值会导致 SQLite 自动生成一个新的唯一主键值。如果尝试将任何其他非整数值插入此列，r-tree 模块会在将其写入数据库之前将其静默转换为整数。

对于"rtree"虚拟表，min/max-value 对列存储为 32 位浮点值，对于"rtree_i32"虚拟表，则存储为 32 位有符号整数。与可以存储各种数据类型和格式的常规 SQLite 表不同，R*Tree 严格执行这些存储类型。如果向这样的列插入任何其他类型的值，r-tree 模块会静默地将其转换为所需的类型，然后将新记录写入数据库。

## 3.1\. 创建 R*Tree 索引

新的 R*Tree 索引的创建如下所示：

```sql
CREATE VIRTUAL TABLE *<name>* USING rtree(*<column-names>*);

```

*<name>*是你的应用为 R*Tree 索引选择的名称，*<column-names>*是一个由 3 到 11 列之间的逗号分隔的列表。虚拟的<name>表创建了三个阴影表来实际存储其内容。这些阴影表的名称分别是：

```sql
*<name>*_node

*<name>*_rowid

*<name>*_parent

```

阴影表是普通的 SQLite 数据表。你可以直接查询它们，但这不太可能会显示出特别有用的内容。你可以 UPDATE、DELETE、INSERT 甚至 DROP 这些阴影表，但这样做会损坏你的 R*Tree 索引。因此最好是简单地忽略阴影表。要认识到它们保存着你的 R*Tree 索引信息，然后就让它们消失。

例如，考虑创建一个用于空间查询的二维 R*Tree 索引：

```sql
CREATE VIRTUAL TABLE demo_index USING rtree(
   id,              -- Integer primary key
   minX, maxX,      -- Minimum and maximum X coordinate
   minY, maxY       -- Minimum and maximum Y coordinate
);

```

### 3.1.1\. 列命名细节

在 CREATE VIRTUAL TABLE 语句中"rtree"的参数中，列的名称取自每个参数的第一个标记。每个参数中的所有后续标记都被静默地忽略。这意味着，例如，如果您尝试给列指定类型亲和性或添加唯一或非 NULL 或默认约束等，这些额外的标记都被视为有效，但它们不会改变 rtree 的行为。在 RTREE 虚拟表中，第一列总是具有整数的类型亲和性，而所有其他数据列都具有实数的类型亲和性。在 RTREE_I32 虚拟表中，所有列都具有整数的类型亲和性。

推荐的做法是在 rtree 规范中省略任何额外的标记。让"rtree"的每个参数都是一个单一的普通标签，即对应列的名称，并删除参数列表中的所有其他标记。

## 3.2\. 填充 R*Tree 索引

通常的 插入，更新，和 删除 命令在 R*树 索引上的工作方式与常规表格相同。因此，要将一些数据插入到我们的示例 R*树 索引中，我们可以像这样做：

```sql
INSERT INTO demo_index VALUES
  (28215, -80.781227, -80.604706, 35.208813, 35.297367),
  (28216, -80.957283, -80.840599, 35.235920, 35.367825),
  (28217, -80.960869, -80.869431, 35.133682, 35.208233),
  (28226, -80.878983, -80.778275, 35.060287, 35.154446),
  (28227, -80.745544, -80.555382, 35.130215, 35.236916),
  (28244, -80.844208, -80.841988, 35.223728, 35.225471),
  (28262, -80.809074, -80.682938, 35.276207, 35.377747),
  (28269, -80.851471, -80.735718, 35.272560, 35.407925),
  (28270, -80.794983, -80.728966, 35.059872, 35.161823),
  (28273, -80.994766, -80.875259, 35.074734, 35.172836),
  (28277, -80.876793, -80.767586, 35.001709, 35.101063),
  (28278, -81.058029, -80.956375, 35.044701, 35.223812),
  (28280, -80.844208, -80.841972, 35.225468, 35.227203),
  (28282, -80.846382, -80.844193, 35.223972, 35.225655);

```

上述条目是夏洛特附近 14 个邮政编码的边界框（经度和纬度）。真实的数据库可能会有成千上万、百万甚至十亿这样的条目，但这个小样本的 14 行足以说明这些概念。

## 3.3\. 查询 R*树 索引

任何有效的查询都可以针对 R*树 索引工作。R*树的实现使某些类型的查询特别高效。对主键的查询是高效的：

```sql
SELECT * FROM demo_index WHERE id=28269;

```

当然，普通的 SQLite 表格也可以有效地对其整数主键进行查询，因此前述内容并不重要。使用 R*树 的主要原因是可以有效地对坐标范围进行范围查询。例如，SQLite 项目的主办公室位于 35.37785, -80.77470。要找出可能为该办公室提供服务的邮政编码，可以这样做：

```sql
SELECT id FROM demo_index
 WHERE minX<=-80.77470 AND maxX>=-80.77470
   AND minY<=35.37785  AND maxY>=35.37785;

```

上述查询将快速定位所有包含 SQLite 主办公室在其边界框内的邮政编码，即使 R*树 包含许多条目。上述是一个“包含”查询的示例。R*树 还支持“重叠”查询。例如，要找出与 28269 邮政编码重叠的所有邮政编码边界框：

```sql
SELECT A.id FROM demo_index AS A, demo_index AS B
 WHERE A.maxX>=B.minX AND A.minX<=B.maxX
   AND A.maxY>=B.minY AND A.minY<=B.maxY
   AND B.id=28269;

```

此第二个查询将找到 28269 条目（因为每个边界框与自身重叠），还会找到其他与 28269 足够接近的邮政编码，它们的边界框也会重叠。

请注意，并不需要 R*树 索引中的所有坐标都受到限制，以便使索引搜索高效。例如，可以查询与北纬 35 度重叠的所有对象：

```sql
SELECT id FROM demo_index
 WHERE maxY>=35.0  AND minY<=35.0;

```

但是，一般来说，R*树 模块需要处理的约束越多，边界框越小，结果返回得越快。

## 3.4\. 精度误差

默认情况下，R*树 中使用 32 位浮点值存储坐标。当一个坐标不能被 32 位浮点数精确表示时，下限坐标会向下舍入，上限坐标会向上舍入。因此，边界框可能会比指定的稍大，但永远不会更小。这正是进行更常见的“重叠”查询所需要的。如果将条目边界框向外舍入，可能会在重叠查询中导致额外的几个条目出现，如果条目边界框的边缘对应于查询边界框的边缘。但重叠查询永远不会漏掉有效的表条目。

但是，对于“包含在内”风格的查询，将边界框向外舍入可能导致某些条目被排除在结果集之外，如果条目边界框的边缘对应于查询边界框的边缘。为了防止这种情况，应用程序应该在每个维度上稍微扩展其“包含在内”查询框（增加 0.000012%），即通过向下舍入较低坐标和向上舍入顶部坐标。

## 3.5\. 同时读取和写入

Guttman R-Tree 算法的特性是，任何写操作可能会彻底重构树，并在此过程中改变节点的扫描顺序。因此，在查询 R-Tree 进行中通常不可能修改 R-Tree。试图这样做将导致一个 SQLITE_LOCKED "数据库表被锁定" 错误。

因此，例如，假设一个应用程序像这样针对 R-Tree 运行一个查询：

```sql
SELECT id FROM demo_index
 WHERE maxY>=35.0  AND minY<=35.0;

```

然后，对于返回的每个“id”值，假设应用程序创建如下 UPDATE 语句，并将返回的“id”值绑定到“?1”参数：

```sql
UPDATE demo_index SET maxY=maxY+0.5 WHERE id=?1;

```

那么更新可能会因 SQLITE_LOCKED 错误而失败。原因是初始查询尚未完成。它记住了在扫描 R-Tree 中间的位置。因此，不能容忍对 R-Tree 的更新，因为这会破坏扫描。

这只是 R-Tree 扩展的一个限制。SQLite 中的普通表能够同时读取和写入。其他虚拟表可能（或可能不会）具有这种能力。在某些情况下，如果可以在开始更新之前可靠地完成查询，则 R-Tree 看起来可以同时读取和写入。但是你不应该期望每个查询都能这样。一般来说，最好避免同时对同一个 R-Tree 运行查询和更新。

如果确实需要根据相同的 R-Tree 的复杂查询更新 R-Tree，最好先运行复杂查询，并将结果存储在临时表中，然后基于存储在临时表中的值更新 R-Tree。

# 4\. 有效使用 R*Trees

对于 SQLite 版本在 3.24.0 之前（2018-06-04），R*Tree 索引存储的关于对象的唯一信息是其整数 ID 和其边界框。额外的信息需要存储在单独的表中，并使用主键与 R*Tree 索引关联。例如，可以创建一个辅助表如下：

```sql
CREATE TABLE demo_data(
  id INTEGER PRIMARY KEY,  -- primary key
  objname TEXT,            -- name of the object
  objtype TEXT,            -- object type
  boundary BLOB            -- detailed boundary of object
);

```

在这个例子中，demo_data.boundary 字段旨在保存对象精确边界的某种二进制表示。R*Tree 索引仅保存对象的轴对齐矩形边界。R*Tree 边界只是真实对象边界的近似。因此，通常的情况是使用 R*Tree 索引缩小搜索范围到候选对象列表，然后对每个候选对象进行更详细和昂贵的计算，以确定候选对象是否真正符合搜索条件。

> **关键点：** R*Tree 索引通常不能提供确切答案，只能将潜在答案集从数百万减少到几十个。

假设 demo_data.boundary 字段保存邮政编码复杂二维边界的某种专有数据描述，并假设应用程序已使用 sqlite3_create_function() 接口创建了一个应用程序定义函数 "contained_in(boundary,lat,long)"，该函数接受 demo_data.boundary 对象以及纬度和经度，并根据边界判断是否返回 true 或 false。可以假设 "contained_in()" 是一个相对较慢的函数，我们不希望频繁调用它。那么，找到主 SQLite 办公室特定邮政编码的有效方法是运行类似于以下的查询：

```sql
SELECT objname FROM demo_data, demo_index
 WHERE demo_data.id=demo_index.id
   AND contained_in(demo_data.boundary, 35.37785, -80.77470)
   AND minX<=-80.77470 AND maxX>=-80.77470
   AND minY<=35.37785  AND maxY>=35.37785;

```

注意上述查询的工作方式：R*Tree 索引在外循环中运行，以查找其边界框中包含 SQLite 主办公室的条目。对于每个找到的行，SQLite 查找 demo_data 表中对应的条目。然后，它使用 demo_data 表中的 boundary 字段作为 contained_in() 函数的参数，如果该函数返回 true，则我们知道所寻找的坐标在该邮政编码边界内。

使用以下更简单的查询，即使没有使用 R*Tree 索引，也能得到相同的答案：

```sql
SELECT objname FROM demo_data
 WHERE contained_in(demo_data.boundary, 35.37785, -80.77470);

```

后一个查询的问题在于必须对 demo_data 表中的所有条目应用 contained_in() 函数。在倒数第二个查询中使用 R*Tree 可将对 contained_in() 函数的调用次数减少到整个表的一个小子集。R*Tree 索引本身并没有找到确切答案，它只是限制了搜索空间。

## 4.1\. 辅助列

从 SQLite 版本 3.24.0（2018-06-04）开始，r-tree 表可以具有存储任意数据的辅助列。辅助列可以代替 "demo_data" 等二级表。

辅助列在列名前标有“+”符号。辅助列必须位于所有坐标边界列之后。一个 RTREE 表最多可以有 100 列。换句话说，包括整数主键列、坐标边界列和所有辅助列在内的列数必须小于或等于 100。下面的示例显示了一个带有辅助列的 r-tree 表，该表相当于上述两个表 "demo_index" 和 "demo_data"：

```sql
CREATE VIRTUAL TABLE demo_index2 USING rtree(
   id,              -- Integer primary key
   minX, maxX,      -- Minimum and maximum X coordinate
   minY, maxY,      -- Minimum and maximum Y coordinate
   +objname TEXT,   -- name of the object
   +objtype TEXT,   -- object type
   +boundary BLOB   -- detailed boundary of object
);

```

将位置数据和相关信息合并到同一表中，辅助列可以提供更清晰的模型并减少连接的需求。例如，之前在 demo_index 和 demo_data 之间的连接 现在可以写成一个简单的查询，如下所示：

```sql
SELECT objname FROM demo_index2
 WHERE contained_in(boundary, 35.37785, -80.77470)
   AND minX<=-80.77470 AND maxX>=-80.77470
   AND minY<=35.37785  AND maxY>=35.37785;

```

### 4.1.1\. 限制

对于辅助列，只有列名是重要的。类型亲和性被忽略。诸如 NOT NULL、UNIQUE、REFERENCES 或 CHECK 的约束也被忽略。然而，SQLite 的未来版本可能会开始关注类型亲和性和约束，因此建议辅助列的用户将其留空，以避免未来的兼容性问题。

# 5\. 整数值 R-Trees

默认的虚拟表（"rtree"）将坐标存储为单精度（4 字节）浮点数。如果需要整数坐标，请使用 "rtree_i32" 声明表：

```sql
CREATE VIRTUAL TABLE intrtree USING rtree_i32(id,x0,x1,y0,y1,z0,z1);

```

一个 rtree_i32 将坐标存储为 32 位带符号整数。尽管它使用整数存储值，但 rtree_i32 虚拟表仍然在 r-tree 算法的内部使用浮点计算。

# 6\. 自定义 R-Tree 查询

通过在 SELECT 查询的 WHERE 子句中使用标准 SQL 表达式，程序员可以查询与特定边界框相交或包含的所有 R*Tree 条目。使用 MATCH 操作符在 SELECT 的 WHERE 子句中进行自定义 R*Tree 查询，允许程序员查询与任意区域或形状相交的 R*Tree 条目集合，而不仅仅是一个框。例如，在计算从位于 3D 空间中的摄像机位置可见的 R*Tree 对象子集时，这种功能非常有用。

自定义 R*Tree 查询的区域由应用程序实现的 R*Tree 几何回调定义，并通过调用以下两个 API 之一向 SQLite 注册：

```sql
int sqlite3_rtree_query_callback(
  sqlite3 *db,
  const char *zQueryFunc,
  int (*xQueryFunc)(sqlite3_rtree_query_info*),
  void *pContext,
  void (*xDestructor)(void*)
);
int sqlite3_rtree_geometry_callback(
  sqlite3 *db,
  const char *zGeom,
  int (*xGeom)(sqlite3_rtree_geometry *, int nCoord, double *aCoord, int *pRes),
  void *pContext
);

```

sqlite3_rtree_query_callback() 在 SQLite 版本 3.8.5（2014-06-04）中可用，是首选接口。sqlite3_rtree_geometry_callback() 是一个较旧且灵活性较差的接口，仅供向后兼容支持。

上述 API 的调用会创建一个以第二个参数（zQueryFunc 或 zGeom）命名的新 SQL 函数。当该 SQL 函数出现在 MATCH 运算符的右侧，并且 MATCH 运算符的左侧是 R*Tree 虚拟表中的任意列时，将调用由第三个参数（xQueryFunc 或 xGeom）定义的回调函数，以确定特定对象或子树是否与所需区域重叠。

例如，像下面这样的查询可能用于查找与圆心位于 (45.3, 22.9)，半径为 5.0 的圆重叠的所有 R*Tree 条目：

```sql
SELECT id FROM demo_index WHERE id MATCH circle(45.3, 22.9, 5.0)

```

无论使用 sqlite3_rtree_geometry_callback() 还是 sqlite3_rtree_query_callback() 接口注册 SQL 函数，自定义查询的 SQL 语法都是相同的。但是，新的查询样式回调允许应用程序更好地控制查询的执行过程。

## 6.1\. 旧版 xGeom 回调

旧版 xGeom 回调使用四个参数调用。第一个参数是指向 sqlite3_rtree_geometry 结构体的指针，提供了有关 SQL 函数调用方式的信息。第二个参数是每个 R*Tree 条目中坐标的数量，对于给定的 R*Tree 总是相同。对于一维 R*Tree，坐标数为 2；对于二维 R*Tree，坐标数为 4；对于三维 R*Tree，坐标数为 6，依此类推。第三个参数 aCoord[] 是一个包含 nCoord 个坐标的数组，定义要测试的边界框。最后一个参数是一个指针，用于写入回调结果。如果由 aCoord[] 定义的边界框完全位于由 xGeom 回调定义的区域外，则结果为零；如果边界框在 xGeom 区域内或重叠，则结果为非零。xGeom 回调通常应返回 SQLITE_OK。如果 xGeom 返回除 SQLITE_OK 之外的任何内容，则 R*Tree 查询将以错误中止。

sqlite3_rtree_geometry 结构体指针作为 xGeom 回调的第一个参数，具体结构如下所示。在同一查询中，对于相同的 MATCH 运算符，每个回调都使用完全相同的 sqlite3_rtree_geometry 结构体。SQLite 初始化了 sqlite3_rtree_geometry 结构体的内容，但后续不再修改。如果需要，回调函数可以自由地更改结构体中的 pUser 和 xDelUser 元素。

```sql
typedef struct sqlite3_rtree_geometry sqlite3_rtree_geometry;
struct sqlite3_rtree_geometry {
  void *pContext;                 /* Copy of pContext passed to s_r_g_c() */
  int nParam;                     /* Size of array aParam */
  double *aParam;                 /* Parameters passed to SQL geom function */
  void *pUser;                    /* Callback implementation user data */
  void (*xDelUser)(void *);       /* Called by SQLite to clean up pUser */
};

```

当注册回调时，sqlite3_rtree_geometry_callback() 中传递的 pContext 参数的副本始终设置为 sqlite3_rtree_geometry 结构体的 pContext 成员。aParam[] 数组（大小为 nParam）包含传递给 MATCH 运算符右侧的 SQL 函数的参数值。在上面的 "circle" 查询示例中，nParam 将设置为 3，并且 aParam[] 数组将包含三个值 45.3, 22.9 和 5.0。

sqlite3_rtree_geometry 结构体的 pUser 和 xDelUser 成员最初被设置为 NULL。回调实现可以将 pUser 变量设置为任何在同一查询中后续调用的回调中有用的任意值（例如，指向用于测试区域交集的复杂数据结构的指针）。如果 xDelUser 变量被设置为非 NULL 值，则在查询运行完毕后 SQLite 会自动以 pUser 变量的值作为唯一参数调用它。换句话说，xDelUser 可以设置为 pUser 值的析构函数。

xGeom 回调总是对 r-tree 执行深度优先搜索。

## 6.2\. 新的 xQueryFunc 回调

新的 xQueryFunc 回调从 r-tree 查询引擎在每次调用时接收更多信息，并在返回之前向查询引擎发送更多信息。为了帮助保持接口的可管理性，xQueryFunc 回调作为 sqlite3_rtree_query_info 结构体中的字段从查询引擎发送和接收信息：

```sql
struct sqlite3_rtree_query_info {
  void *pContext;                   /* pContext from when function registered */
  int nParam;                       /* Number of function parameters */
  sqlite3_rtree_dbl *aParam;        /* value of function parameters */
  void *pUser;                      /* callback can use this, if desired */
  void (*xDelUser)(void*);          /* function to free pUser */
  sqlite3_rtree_dbl *aCoord;        /* Coordinates of node or entry to check */
  unsigned int *anQueue;            /* Number of pending entries in the queue */
  int nCoord;                       /* Number of coordinates */
  int iLevel;                       /* Level of current node or entry */
  int mxLevel;                      /* The largest iLevel value in the tree */
  sqlite3_int64 iRowid;             /* Rowid for current entry */
  sqlite3_rtree_dbl rParentScore;   /* Score of parent node */
  int eParentWithin;                /* Visibility of parent node */
  int eWithin;                      /* OUT: Visiblity */
  sqlite3_rtree_dbl rScore;         /* OUT: Write the score here */
  /* The following fields are only available in 3.8.11 and later */
  sqlite3_value **apSqlParam;       /* Original SQL values of parameters */
};

```

sqlite3_rtree_query_info 结构体的前五个字段与 sqlite3_rtree_geometry 结构体相同，并具有完全相同的含义。sqlite3_rtree_query_info 结构体还包含 nCoord 和 aCoord 字段，它们具有与 xGeom 回调中同名参数完全相同的含义。

xQueryFunc 必须将 sqlite3_rtree_query_info 的 eWithin 字段设置为 NOT_WITHIN、PARTLY_WITHIN 或 FULLY_WITHIN 值之一，具体取决于 aCoord[]定义的边界框是完全在区域外、与区域重叠还是完全在区域内。此外，xQueryFunc 必须将 rScore 字段设置为非负值，指示查询中应分析和返回子树和条目的顺序。较小的分数首先处理。

正如其名，R*树被组织成一棵树。树的每个节点都是一个边界框。树的根是一个边界框，它封装了树的所有元素。在根节点下面有若干个子树（通常是 20 个或更多），每个子树都有自己的较小边界框，并且每个子树包含 R*树条目的某个子集。子树可能还有子子树，以此类推，直到最后到达树的叶子节点，这些叶子节点就是实际的 R*树条目。

R*Tree 查询通过将根节点作为按 rScore 排序的优先队列中的唯一条目来初始化。查询通过提取优先队列中具有最低分数的条目来进行。如果该条目是叶子节点（表示它是实际的 R*Tree 条目而不是子树），则将该条目作为查询结果的一行返回。如果提取的优先队列条目是一个节点（一个子树），则将该节点的下一个子节点传递给 xQueryFunc 回调。如果节点有更多子节点，则将其返回到优先队列。否则将其丢弃。xQueryFunc 回调设置 eWithin 为 PARTLY_WITHIN 或 FULLY_WITHIN 的子元素将使用回调提供的分数添加到优先队列中。返回 NOT_WITHIN 的子元素将被丢弃。查询在优先队列为空时结束。

R*Tree 中的每个叶子条目和节点（子树）都有一个整数“级别”。叶子的级别为 0。叶子的第一个包含子树的级别为 1。R*Tree 的根具有最大的级别值。sqlite3_rtree_query_info 结构中的 mxLevel 条目是 R*Tree 根的级别值。sqlite3_rtree_query_info 中的 iLevel 条目给出了正在查询的对象的级别。

大多数 R*Tree 查询使用深度优先搜索。通过将 rScore 设置为 iLevel 来实现此操作。通常偏爱深度优先搜索，因为它可以最小化优先队列中的元素数量，从而降低内存需求并加快处理速度。然而，某些应用程序可能更喜欢广度优先搜索，可以通过将 rScore 设置为 mxLevel-iLevel 来实现。通过为 rScore 创建更复杂的公式，应用程序可以详细控制搜索子树和返回叶 R*Tree 条目的顺序。例如，在具有数百万 R*Tree 条目的应用程序中，可以安排 rScore，以便首先返回最大或最重要的条目，从而使应用程序能够快速显示最重要的信息，并在可用时填充更小和不太重要的细节。

如果需要，sqlite3_rtree_query_info 结构的其他信息字段可以由 xQueryFunc 回调使用。iRowid 字段是正在考虑的元素的行号（R*Tree 中的 3 到 11 列中的第一个）。iRowid 仅对叶子有效。eParentWithin 和 rParentScore 值是当前行所在子树的 eWithin 和 rScore 值的副本。anQueue 字段是一个 mxLevel+1 个无符号整数的数组，告知当前每个级别优先队列中的元素数量。

## 6.3\. 自定义查询的其他注意事项

自定义 R*Tree 查询函数的 MATCH 运算符必须是 WHERE 子句的顶级 AND 连接术语，否则将无法由 R*Tree 查询优化器使用，查询将无法运行。例如，如果 MATCH 运算符通过 OR 运算符连接到 WHERE 子句的其他术语，则查询将失败并显示错误。

同一 WHERE 子句中允许使用两个或多个 MATCH 运算符，只要它们通过 AND 运算符连接即可。但是，R*Tree 查询引擎仅包含单个优先级队列。在搜索中分配给每个节点的优先级是由任何 MATCH 运算符返回的最低优先级。

# 实现细节。

下面的部分描述了 R*Tree 实现的一些低级细节，这对故障排除或性能分析可能有用。

## 7.1\. 影子表

R*Tree 索引的内容实际上存储在三个普通的 SQLite 表中，这些表的名称源自 R*Tree 的名称。这三个表称为"影子表"。它们的架构如下：

```sql
CREATE TABLE %_node(nodeno INTEGER PRIMARY KEY, data)
CREATE TABLE %_parent(nodeno INTEGER PRIMARY KEY, parentnode)
CREATE TABLE %_rowid(rowid INTEGER PRIMARY KEY, nodeno)

```

每个影子表的名称中的"%"被替换为 R*Tree 虚拟表的名称。因此，如果 R*Tree 表的名称是"xyz"，那么这三个影子表将是"xyz_node"，"xyz_parent"和"xyz_rowid"。

%_node 表中每个 R*Tree 节点都有一个条目。R*Tree 节点由互相接近的一个或多个条目组成。对于树的 R*Tree，除了根节点之外的所有节点在%_parent 影子表中都有一个条目，用于标识父节点。R*Tree 的每个条目都有一个 rowid。%_rowid 影子表将条目 rowid 映射到包含该条目的节点。

附加到%_rowid 表的额外列保存辅助列的内容。这些额外的%_rowid 列的名称可能与实际的辅助列名称不同。

## 使用 rtreecheck() SQL 函数进行完整性检查。

标量 SQL 函数 rtreecheck(R)或 rtreecheck(S,R)对数据库 S 中包含的名为 R 的 rtree 表运行完整性检查。该函数返回发现的任何问题的人类语言描述，或者如果一切正常，则返回字符串'ok'。在 R*Tree 虚拟表上运行 rtreecheck()类似于在数据库上运行 PRAGMA integrity_check。

示例：要验证名为"demo_index"的 R*Tree 是否形成良好且内部一致，请运行：

```sql
SELECT rtreecheck('demo_index');

```

rtreecheck()函数执行以下检查：

1.  对于 rtree 结构（%_node 表）中的每个单元格：

    1.  对于每个维度，（coord1 <= coord2）。

    1.  除非单元格位于根节点上，否则该单元格由父节点上的父单元格限定。

    1.  对于叶节点，%_rowid 表中有一个条目，其对应于指向正确节点的单元格的 rowid 值。

    1.  对于非叶子节点上的单元格，%_parent 表中有一个条目，将单元格的子节点映射到其所在的节点。

1.  在 %_rowid 表中的条目数与 r 树结构中的叶子单元格数相同，并且每个 %_rowid 表条目对应一个叶子单元格。

1.  在 %_parent 表中的条目数与 r 树结构中的非叶子单元格数相同，并且每个 %_parent 表条目对应一个非叶子单元格。
