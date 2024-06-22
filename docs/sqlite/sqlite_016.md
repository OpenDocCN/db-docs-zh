# 由 SQLite 理解的 SQL 语言

> 原文：[`sqlite.com/lang.html`](https://sqlite.com/lang.html)

SQLite 理解大多数标准 SQL 语言。但它在某些功能上进行了省略，同时添加了一些自己的功能。本文试图准确描述 SQLite 支持和不支持的 SQL 语言的部分。还提供了一个 SQL 关键字的列表。SQL 语言的语法由语法图描述。

可用以下语法文档主题：

|

+   聚合函数

+   修改表

+   分析

+   连接数据库

+   开始事务

+   评论

+   提交事务

+   核心函数

+   创建索引

+   创建表

+   创建触发器

+   创建视图

+   创建虚拟表

+   日期和时间函数

+   删除

+   分离数据库

+   删除索引

+   删除表

+   删除触发器

+   删除视图

+   结束事务

+   解释

+   表达式

+   INDEXED BY

+   插入

+   JSON 函数

+   关键字

+   数学函数

+   ON CONFLICT 子句

+   PRAGMA

+   重新索引

+   释放保存点

+   替换

+   RETURNING 子句

+   回滚事务

+   保存点

+   选择

+   更新

+   UPSERT

+   清理

+   窗口函数

+   WITH 子句

|

例程 sqlite3_prepare_v2()，sqlite3_prepare()，sqlite3_prepare16()，sqlite3_prepare16_v2()，sqlite3_exec()，以及 sqlite3_get_table()接受一个 SQL 语句列表（sql-stmt-list），这是一个由分号分隔的语句列表。

**sql-stmt-list:**

<svg class="pikchr" viewBox="0 0 242.093 88.776"><text x="121" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">sql-stmt</text> <text x="121" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">;</text></svg>

每个语句列表中的 SQL 语句是以下内容的一个实例：

**sql-stmt:**

<svg class="pikchr" viewBox="0 0 716.88 1017.36"><text x="95" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">解释</text> <text x="213" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">查询</text> <text x="297" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">计划</text> <text x="481" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">alter-table-stmt</text> <text x="469" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分析-stmt</text> <text x="463" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">附加-stmt</text> <text x="460" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">开始-stmt</text> <text x="468" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">提交-stmt</text> <text x="489" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">创建索引-stmt</text> <text x="488" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">创建表-stmt</text> <text x="496" y="281" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">创建触发器-stmt</text> <text x="486" y="319" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">创建视图-stmt</text> <text x="518" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">创建虚拟表-stmt</text> <text x="463" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">删除-stmt</text> <text x="495" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有限删除-stmt</text> <text x="465" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分离-stmt</text> <text x="482" y="508" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">删除索引-stmt</text> <text x="480" y="546" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">删除表-stmt</text> <text x="489" y="584" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">删除触发器-stmt</text> <text x="478" y="622" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">删除视图-stmt</text> <text x="461" y="659" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">插入-stmt</text> <text x="469" y="697" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">编译指令-stmt</text> <text x="469" y="735" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">重建指令-stmt</text> <text x="468" y="773" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">释放指令-stmt</text> <text x="471" y="811" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">回滚指令-stmt</text> <text x="477" y="848" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">保存点-stmt</text> <text x="462" y="886" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">选择-stmt</text> <text x="466" y="924" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">更新-stmt</text> <text x="498" y="962" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有限更新-stmt</text> <text x="469" y="1000" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">真空-stmt</text></svg>
