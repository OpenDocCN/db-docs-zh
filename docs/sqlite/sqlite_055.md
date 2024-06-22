# SQLite 与其他数据库引擎中的 NULL 处理比较

> 原文：[`sqlite.com/nulls.html`](https://sqlite.com/nulls.html)

目标是使 SQLite 以符合标准的方式处理 NULL。但是，SQL 标准文档中关于如何处理 NULL 的描述似乎存在歧义。从标准文档中并不清楚在所有情况下应如何处理 NULL。

因此，与其依照标准文档，不如测试各种流行的 SQL 引擎以查看它们如何处理 NULL。这个想法是让 SQLite 像所有其他引擎一样工作。开发了一个 SQL 测试脚本，并由志愿者在各种 SQL RDBMS 上运行，并使用这些测试结果推断每个引擎如何处理 NULL 值。最初的测试是在 2002 年 5 月进行的。测试脚本的副本可在本文档的末尾找到。

SQLite 最初的编码方式是所有表格下所有问题的答案都是“是”。但是在其他 SQL 引擎上运行的实验显示，它们都不起作用。因此，SQLite 被修改为与 Oracle、PostgreSQL 和 DB2 一样工作。这涉及使 NULL 在 SELECT DISTINCT 语句和 SELECT 中的 UNION 运算符中不明显。在唯一列中，NULL 仍然是独特的。这似乎有些武断，但与其他引擎兼容的愿望胜过了这一反对意见。

可以让 SQLite 在 SELECT DISTINCT 和 UNION 的情况下将 NULL 视为独特。为此，应修改 `sqliteInt.h` 源文件中的 NULL_ALWAYS_DISTINCT #define 的值，并重新编译。

> *更新 2003-07-13：* 由于本文档最初编写时测试过的某些数据库引擎已经更新，并且用户们也很友好地向下面的表格发送了更正意见。原始数据显示了各种行为，但随着时间推移，行为范围已经收敛到了 PostgreSQL/Oracle 模型。唯一的显著差异是 Informix 和 MS-SQL 在唯一列中将 NULL 视为不明显。
> 
> NULL 在唯一列中独特，但在 SELECT DISTINCT 和 UNION 中不明显的事实仍然令人困惑。似乎 NULL 应该在所有地方都是独特的或者都不独特。SQL 标准文档建议 NULL 应该在所有情况下都是独特的。然而，截至本文撰写时，没有测试的 SQL 引擎将 NULL 视为在 SELECT DISTINCT 语句或 UNION 中是独特的。

下表显示了 NULL 处理实验的结果。

|    | SQLite | PostgreSQL | Oracle | Informix | DB2 | MS-SQL | OCELOT |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 加任何值到 NULL 得到 NULL | 是 | 是 | 是 | 是 | 是 | 是 | 是 |
| 将 NULL 乘以零得到 NULL | 是 | 是 | 是 | 是 | 是 | 是 | 是 |
| 在唯一列中，NULL 是独特的 | 是 | 是 | 是 | 否 | (注释 4) | 否 | 是 |
| 在 SELECT DISTINCT 中，NULL 是独特的 | 否 | 否 | 否 | 否 | 否 | 否 | 否 |
| 在 UNION 中，NULL 是独特的 | 否 | 否 | 否 | 否 | 否 | 否 | 否 |
| "CASE WHEN null THEN 1 ELSE 0 END" is 0? | 是 | 是 | 是 | 是 | 是 | 是 | 是 |
| "null OR true" is true | 是 | 是 | 是 | 是 | 是 | 是 | 是 |
| "not (null AND false)" is true | 是 | 是 | 是 | 是 | 是 | 是 | 是 |
|    | MySQL 3.23.41 | MySQL 4.0.16 | Firebird | SQL Anywhere | Borland Interbase |
| Adding anything to null gives null | 是 | 是 | 是 | 是 | 是 |
| Multiplying null by zero gives null | 是 | 是 | 是 | 是 | 是 |
| nulls are distinct in a UNIQUE column | 是 | 是 | 是 | (注 4) | (注 4) |
| nulls are distinct in SELECT DISTINCT | 否 | 否 | 否 (注 1) | 否 | 否 |
| nulls are distinct in a UNION | (注 3) | 否 | 否 (注 1) | 否 | 否 |
| "CASE WHEN null THEN 1 ELSE 0 END" is 0? | 是 | 是 | 是 | 是 | (注 5) |
| "null OR true" is true | 是 | 是 | 是 | 是 | 是 |
| "not (null AND false)" is true | 否 | 是 | 是 | 是 | 是 |
| Notes:   | 1.  | Firebird 的旧版本从 SELECT DISTINCT 和 UNION 中省略所有 NULL。 |
| 2.  | 测试数据不可用。 |
| 3.  | MySQL 版本 3.23.41 不支持 UNION。 |
| 4.  | DB2、SQL Anywhere 和 Borland Interbase 不允许在唯一列中使用 NULL。 |
| 5.  | Borland Interbase 不支持 CASE 表达式。 |

以下脚本用于收集上表的信息。

```sql
-- I have about decided that SQL's treatment of NULLs is capricious and cannot be
-- deduced by logic.  It must be discovered by experiment.  To that end, I have 
-- prepared the following script to test how various SQL databases deal with NULL.
-- My aim is to use the information gathered from this script to make SQLite as
-- much like other databases as possible.
--
-- If you could please run this script in your database engine and mail the results
-- to me at drh@hwaci.com, that will be a big help.  Please be sure to identify the
-- database engine you use for this test.  Thanks.
--
-- If you have to change anything to get this script to run with your database
-- engine, please send your revised script together with your results.
--

-- Create a test table with data
create table t1(a int, b int, c int);
insert into t1 values(1,0,0);
insert into t1 values(2,0,1);
insert into t1 values(3,1,0);
insert into t1 values(4,1,1);
insert into t1 values(5,null,0);
insert into t1 values(6,null,1);
insert into t1 values(7,null,null);

-- Check to see what CASE does with NULLs in its test expressions
select a, case when b<>0 then 1 else 0 end from t1;
select a+10, case when not b<>0 then 1 else 0 end from t1;
select a+20, case when b<>0 and c<>0 then 1 else 0 end from t1;
select a+30, case when not (b<>0 and c<>0) then 1 else 0 end from t1;
select a+40, case when b<>0 or c<>0 then 1 else 0 end from t1;
select a+50, case when not (b<>0 or c<>0) then 1 else 0 end from t1;
select a+60, case b when c then 1 else 0 end from t1;
select a+70, case c when b then 1 else 0 end from t1;

-- What happens when you multiply a NULL by zero?
select a+80, b*0 from t1;
select a+90, b*c from t1;

-- What happens to NULL for other operators?
select a+100, b+c from t1;

-- Test the treatment of aggregate operators
select count(*), count(b), sum(b), avg(b), min(b), max(b) from t1;

-- Check the behavior of NULLs in WHERE clauses
select a+110 from t1 where b<10;
select a+120 from t1 where not b>10;
select a+130 from t1 where b<10 OR c=1;
select a+140 from t1 where b<10 AND c=1;
select a+150 from t1 where not (b<10 AND c=1);
select a+160 from t1 where not (c=1 AND b<10);

-- Check the behavior of NULLs in a DISTINCT query
select distinct b from t1;

-- Check the behavior of NULLs in a UNION query
select b from t1 union select b from t1;

-- Create a new table with a unique column.  Check to see if NULLs are considered
-- to be distinct.
create table t2(a int, b int unique);
insert into t2 values(1,1);
insert into t2 values(2,null);
insert into t2 values(3,null);
select * from t2;

drop table t1;
drop table t2;

```
