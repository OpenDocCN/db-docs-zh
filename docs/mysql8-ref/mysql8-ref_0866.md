# 14.19.4 功能依赖的检测

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html`](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html)

以下讨论提供了 MySQL 检测功能依赖的几个示例。示例使用以下符号表示：

```sql
{*X*} -> {*Y*}
```

将其理解为“*`X`* 唯一确定 *`Y`*”，这也意味着 *`Y`* 在 *`X`* 上是函数依赖的。

示例使用 `world` 数据库，可以从 `dev.mysql.com/doc/index-other.html` 下载。您可以在同一页面找到如何安装数据库的详细信息。

+   [从键派生的功能依赖](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html#functional-dependence-keys "从键派生的功能依赖")

+   [从多列键和等式派生的功能依赖](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html#functional-dependence-multiple-column-keys "从多列键和等式派生的功能依赖")

+   [功能依赖特殊情况](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html#functional-dependence-special-cases "功能依赖特殊情况")

+   [功能依赖和视图](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html#functional-dependence-views "功能依赖和视图")

+   [功能依赖的组合](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html#functional-dependence-combinations "功能依赖的组合")

#### 从键派生的功能依赖

以下查询为每个国家选择使用语言的人数：

```sql
SELECT co.Name, COUNT(*)
FROM countrylanguage cl, country co
WHERE cl.CountryCode = co.Code
GROUP BY co.Code;
```

`co.Code` 是 `co` 的主键，因此 `co` 的所有列都对其具有函数依赖，使用以下符号表示：

```sql
{co.Code} -> {co.*}
```

因此，`co.name` 在 `GROUP BY` 列上是函数依赖的，查询是有效的。

可以使用在 `NOT NULL` 列上的 `UNIQUE` 索引代替主键，相同的功能依赖也适用。（对于允许 `NULL` 值的 `UNIQUE` 索引，这不成立，因为它允许多个 `NULL` 值，此时唯一性丢失。）

#### 从多列键和等式派生的功能依赖

此查询为每个国家选择所有使用的语言及使用该语言的人数列表：

```sql
SELECT co.Name, cl.Language,
cl.Percentage * co.Population / 100.0 AS SpokenBy
FROM countrylanguage cl, country co
WHERE cl.CountryCode = co.Code
GROUP BY cl.CountryCode, cl.Language;
```

(`cl.CountryCode`, `cl.Language`) 是 `cl` 的两列复合主键，因此列对唯一确定了 `cl` 的所有列：

```sql
{cl.CountryCode, cl.Language} -> {cl.*}
```

此外，由于 `WHERE` 子句中的等式：

```sql
{cl.CountryCode} -> {co.Code}
```

并且，因为 `co.Code` 是 `co` 的主键：

```sql
{co.Code} -> {co.*}
```

“唯一确定”关系是传递的，因此：

```sql
{cl.CountryCode, cl.Language} -> {cl.*,co.*}
```

结果，查询是有效的。

与前面的示例一样，可以使用在 `NOT NULL` 列上的 `UNIQUE` 键代替主键。

可以使用 `INNER JOIN` 条件代替 `WHERE`。相同的功能依赖适用：

```sql
SELECT co.Name, cl.Language,
cl.Percentage * co.Population/100.0 AS SpokenBy
FROM countrylanguage cl INNER JOIN country co
ON cl.CountryCode = co.Code
GROUP BY cl.CountryCode, cl.Language;
```

#### 功能依赖特殊情况

而在`WHERE`条件或`INNER JOIN`条件中的相等性测试是对称的，但在外连接条件中的相等性测试不是，因为表扮演不同的角色。

假设引用完整性被意外破坏，并且存在一个`countrylanguage`中没有对应行的`country`行。考虑与前一个示例中相同的查询，但使用`LEFT JOIN`：

```sql
SELECT co.Name, cl.Language,
cl.Percentage * co.Population/100.0 AS SpokenBy
FROM countrylanguage cl LEFT JOIN country co
ON cl.CountryCode = co.Code
GROUP BY cl.CountryCode, cl.Language;
```

对于给定的`cl.CountryCode`值，在连接结果中`co.Code`的值要么在匹配行中找到（由`cl.CountryCode`确定），要么如果没有匹配则是`NULL`-补充的（也由`cl.CountryCode`确定）。在每种情况下，这种关系适用：

```sql
{cl.CountryCode} -> {co.Code}
```

`cl.CountryCode`本身对{`cl.CountryCode`，`cl.Language`}具有函数依赖，这是一个主键。

如果在连接结果中`co.Code`是`NULL`-补充的，那么`co.Name`也是。如果`co.Code`没有被`NULL`-补充，那么因为`co.Code`是主键，它决定了`co.Name`。因此，在所有情况下：

```sql
{co.Code} -> {co.Name}
```

产生：

```sql
{cl.CountryCode, cl.Language} -> {cl.*,co.*}
```

结果，该查询是有效的。

然而，假设表被交换，如此查询：

```sql
SELECT co.Name, cl.Language,
cl.Percentage * co.Population/100.0 AS SpokenBy
FROM country co LEFT JOIN countrylanguage cl
ON cl.CountryCode = co.Code
GROUP BY cl.CountryCode, cl.Language;
```

现在这种关系*不*适用：

```sql
{cl.CountryCode, cl.Language} -> {cl.*,co.*}
```

实际上，为`cl`制作的所有`NULL`-补充行被放入一个单独的组中（它们的`GROUP BY`列都等于`NULL`），在这个组内，`co.Name`的值可以变化。查询是无效的，MySQL 拒绝它。

外连接中的函数依赖因此与决定性列属于`LEFT JOIN`的左侧还是右侧有关。如果存在嵌套的外连接或连接条件不完全由相等比较组成，则函数依赖的确定变得更加复杂。

#### 函数依赖和视图

假设一个关于国家的视图生成它们的代码、它们的大写名称以及它们拥有多少种不同的官方语言：

```sql
CREATE VIEW country2 AS
SELECT co.Code, UPPER(co.Name) AS UpperName,
COUNT(cl.Language) AS OfficialLanguages
FROM country AS co JOIN countrylanguage AS cl
ON cl.CountryCode = co.Code
WHERE cl.isOfficial = 'T'
GROUP BY co.Code;
```

这个定义是有效的，因为：

```sql
{co.Code} -> {co.*}
```

在视图结果中，第一个选择的列是`co.Code`，它也是分组列，因此决定了所有其他选择的表达式：

```sql
{country2.Code} -> {country2.*}
```

MySQL 理解这一点并使用这些信息，如下所述。

该查询显示了国家、它们拥有多少种不同的官方语言以及它们拥有多少个城市，通过将视图与`city`表进行连接：

```sql
SELECT co2.Code, co2.UpperName, co2.OfficialLanguages,
COUNT(*) AS Cities
FROM country2 AS co2 JOIN city ci
ON ci.CountryCode = co2.Code
GROUP BY co2.Code;
```

这个查询是有效的，因为如前所述：

```sql
{co2.Code} -> {co2.*}
```

MySQL 能够发现视图结果中的函数依赖，并使用它来验证使用该视图的查询。如果`country2`是一个派生表（或公共表达式），情况也是如此，如下所示：

```sql
SELECT co2.Code, co2.UpperName, co2.OfficialLanguages,
COUNT(*) AS Cities
FROM
(
 SELECT co.Code, UPPER(co.Name) AS UpperName,
 COUNT(cl.Language) AS OfficialLanguages
 FROM country AS co JOIN countrylanguage AS cl
 ON cl.CountryCode=co.Code
 WHERE cl.isOfficial='T'
 GROUP BY co.Code
) AS co2
JOIN city ci ON ci.CountryCode = co2.Code
GROUP BY co2.Code;
```

#### 函数依赖的组合

MySQL 能够结合所有前述类型的函数依赖（基于键、基于相等性、基于视图）来验证更复杂的查询。
