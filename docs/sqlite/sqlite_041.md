# 1\. 概述

> 原文：[`sqlite.com/spellfix1.html`](https://sqlite.com/spellfix1.html)

这个 spellfix1 虚拟表 可以用来搜索大词汇量中的近似匹配项。例如，spellfix1 可以用来建议修正拼写错误的单词。或者，它可以与 FTS4 一起使用，用于使用可能拼写错误的单词进行全文搜索。

spellfix1 虚拟表的实现存储在 SQLite 源代码树的 miscellaneous extensions 文件夹中，特别是在文件[ext/misc/spellfix1.c](https://www.sqlite.org/src/finfo?name=ext/misc/spellfix.c)中。spellfix1 虚拟表未包含在 SQLite amalgamation 中，也不是任何标准 SQLite 构建的一部分。它是一个可加载扩展。

加载 spellfix1 扩展后，可以像这样创建一个 spellfix1 虚拟表的实例：

> ```sql
> CREATE VIRTUAL TABLE demo USING spellfix1;
> 
> ```

"spellfix1"术语是 spellfix 模块的名称，必须按照所示输入。"demo"术语是您将创建的虚拟表的名称，可以根据应用程序的需求进行更改。虚拟表最初为空的。为了使虚拟表有用，您需要使用词汇填充它。假设您有一个名为"big_vocabulary"的单词列表表。然后执行以下操作：

> ```sql
> INSERT INTO demo(word) SELECT word FROM big_vocabulary;
> 
> ```

如果您打算与 FTS4 表合作使用此虚拟表（用于搜索词语的拼写更正），则可以使用 fts4aux 表提取词汇：

> ```sql
> INSERT INTO demo(word) SELECT term FROM search_aux WHERE col='*';
> 
> ```

您还可以为每个单词提供一个"rank"给虚拟表。"rank"是单词常见程度的估计值。较大的数字表示单词更常见。如果在填充表时省略 rank，则假定 rank 为 1。但是，如果有 rank 信息，您可以提供它，虚拟表将倾向于选择更常用的术语。要从 fts4aux 表"search_aux"填充 rank，可以执行以下操作：

> ```sql
> INSERT INTO demo(word,rank)
>    SELECT term, documents FROM search_aux WHERE col='*';
> 
> ```

要查询虚拟表，请在 WHERE 子句中包含 MATCH 操作符。例如：

> ```sql
> SELECT word FROM demo WHERE word MATCH 'kennasaw';
> 
> ```

使用美国地名数据集（源自[`geonames.usgs.gov/domestic/download_data.htm`](http://geonames.usgs.gov/domestic/download_data.htm)），上述查询返回以以下方式开始的 20 个结果：

> ```sql
> kennesaw
> kenosha
> kenesaw
> kenaga
> keanak
> 
> ```

如果您在模式末尾附加字符'*'，则执行前缀搜索。例如：

> ```sql
> SELECT word FROM demo WHERE word MATCH 'kennes*';
> 
> ```

返回以以下方式开始的 20 个结果：

> ```sql
> kennesaw
> kennestone
> kenneson
> kenneys
> keanes
> keenes
> 
> ```

# 2\. 搜索精炼

默认情况下，spellfix1 表最多返回 20 个结果。（如果匹配较少，则可能返回少于 20 个。）您可以通过在查询的 WHERE 子句中添加"top=N"项来更改返回行数的上限，其中 N 是新的最大值。例如，要查看前 5 个最佳匹配：

> ```sql
> SELECT word FROM demo WHERE word MATCH 'kennes*' AND top=5;
> 
> ```

spellfix1 虚拟表中的每个条目都与特定的语言相关联，由整数 "langid" 列标识。默认 langid 为 0，如果没有其他操作，则整个词汇表是 0 语言的一部分。但如果您的应用程序需要在多种语言中运行，则可以在填充表时通过指定 langid 字段来为每种语言指定不同的词汇项。例如：

> ```sql
> INSERT INTO demo(word,langid) SELECT word, 0 FROM en_vocabulary;
> INSERT INTO demo(word,langid) SELECT word, 1 FROM de_vocabulary;
> INSERT INTO demo(word,langid) SELECT word, 2 FROM fr_vocabulary;
> INSERT INTO demo(word,langid) SELECT word, 3 FROM ru_vocabulary;
> INSERT INTO demo(word,langid) SELECT word, 4 FROM cn_vocabulary;
> 
> ```

在虚拟表从多种语言中填充项目后，可以在查询的 WHERE 子句中使用 "langid=N" 项来指定感兴趣的语言：

> ```sql
> SELECT word FROM demo WHERE word MATCH 'hildes*' AND langid=1;
> 
> ```

注意，如果在 WHERE 子句中不包括 "langid=N" 项，则搜索将针对语言 0（如上例中的英语）。所有 spellfix1 搜索都针对单个语言 ID。无法同时搜索所有语言。

# 3\. 虚拟表详情

spellfix1 虚拟表中的每一行都有一个唯一的 rowid，具有七列加上五个额外隐藏列。列如下：

**rowid**

与表中每个词汇项关联的唯一整数号码。这可以在数据库中的其他表上用作外键。

**word**

与模式匹配的单词的文本。单词和模式都可以包含 Unicode 字符，并且可以是混合大小写。

**rank**

这是单词的排名，如原始的 INSERT 语句中所指定的。

**distance**

这是从模式到单词的编辑距离或 Levenshtein 距离。

**langid**

这是单词的语言 ID。所有查询都针对单个语言 ID，默认为 0。对于任何给定的查询，该值在所有行上是相同的。

**score**

分数是排名和距离的组合。理念是分数越低越好。虚拟表尝试找到分数最低的单词，默认情况下（除非通过 ORDER BY 覆盖），按照分数递增的顺序返回结果。

**matchlen**

在前缀搜索中，matchlen 是与前缀匹配的字符串中的字符数。对于非前缀搜索，这与 word 的长度相同。

**phonehash**

此列显示用于限制搜索的音标哈希前缀。对于任何给定的查询，此列对于每一行应该是相同的。此信息可用于诊断目的，通常不被认为在实际应用中有用。

**top**

（隐藏）对于任何查询，该值在所有行上是相同的。它是一个整数，表示将输出的最大行数。实际输出的行数可能少于此数，但绝不会更多。top 的默认值为 20，但可以通过在查询的 WHERE 子句中包括形式为 "top=N" 的项来更改每个查询的值。

**scope**

(隐藏) 对于任何查询，该值在所有行中都是相同的。范围是对虚拟表如何广泛地寻找匹配词的一个衡量标准。范围的较小值会导致更广泛的搜索。范围通常会自动选择，并且最大为 4。应用程序可以通过在查询的 WHERE 子句中包含一个形式为“scope=N”的术语来更改范围。增加范围将使查询运行更快，但会减少可能的更正。

**srchcnt**

(隐藏) 对于任何查询，该值在所有行中都是相同的。该值是一个整数，是使用编辑距离算法检查的单词数量，以找到最终显示的最佳匹配。此值仅用于诊断目的。

**soundslike**

(隐藏) 在插入词汇条目时，可以将此字段设置为与单词发音相符的拼写。有关详细信息，请参见下面的处理与不寻常和困难的拼写部分。

**command**

(隐藏) “command”列的值始终为 NULL。但是，应用程序可以在“command”列中插入特殊字符串，以引发拼写修复 1 虚拟表中的某些行为。例如，将字符串“reset”插入到“command”列中将导致虚拟表重新读取其编辑距离权重（如果有）。

# 4\. 算法

拼写修复 1 虚拟表创建一个名为“%_vocab”（其中%被虚拟表的名称替换；例如：“demo”虚拟表的“demo_vocab”）的单个影子表。影子表包含以下列：

**id**

唯一 ID（INTEGER PRIMARY KEY）

**rank**

词的排名。

**langid**

此条目的语言 ID。

**word**

词汇单词的原始 UTF8 文本

**k1**

将单词音译为小写 ASCII。有一张标准表格，可以将非 ASCII 字符映射为 ASCII。例如：“æ”->“ae”，“þ”->“th”，“ß”->“ss”，“á”->“a”，……辅助函数 spellfix1_translit(X)将执行非 ASCII 到 ASCII 映射。内置的 lower(X)函数将转换为小写。因此：k1= lower(spellfix1_translit(word))。如果单词已经全部是小写 ASCII，则 k1 列将包含 NULL。这可以减少%_vocab 表的存储要求，并有助于 spellfix 运行稍快。因此，有利的是尽可能使用小写 ASCII 词汇填充拼写修复表。

**k2**

此字段包含从 coalesce(k1,word)派生的音标代码。具有相似发音的字母被映射为相同的符号。例如，所有元音和元音组成变成单个符号“A”。字母“p”、“b”、“f”和“v”都变成“B”。所有鼻音都表示为“N”。依此类推。映射基于 Soundex、Metaphone 和其他长期存在的音标匹配系统中的想法。此键可以由函数 spellfix1_phonehash(X)生成。因此：k2 = spellfix1_phonehash(coalesce(k1,word))

这里还有一个用于计算模式和单词之间的瓦格纳编辑距离或莱文斯坦距离的函数。这个函数通过 `spellfix1_editdist(X,Y)` 暴露出来。编辑距离函数返回将 X 转换为 Y 的“成本”。有些转换比其他转换更昂贵。例如，将一个元音改为另一个元音相对较便宜，如同一个常数，或省略双常数的第二个字符。其他的转换则更昂贵。编辑距离函数的思想是，对于相似的词返回一个低成本，对于相距较远的词返回一个较高的成本。在这个实现中，任何单字符编辑（删除、插入或替换）的最大成本是 100，某些编辑的成本更低（如变换元音）。

比较的 "分数" 是模式和单词之间的编辑距离，通过单词排名的基数对数调整。例如，距离为 100 但排名为 1000 的匹配具有分数 122（= 100 - log2(1000) + 32），而距离为 100 但排名为 1 的匹配具有分数 131（100 - log2(1) + 32）。通过这种方式，经常使用的单词得到稍低的成本，这倾向于将它们移动到替代拼写列表的顶部。

拼写校正器的一个简单实现是将搜索术语与词汇表中的每个单词进行比较，并选择最低分的前 20 个。但是，词汇表中通常会有数十万或数百万个单词，因此这种方法不够快速。

假设被拼写纠正的术语是 X。为了限制搜索空间，X 被转换为一个类似于 k2 的关键字，使用等效于：

> ```sql
>    key = spellfix1_phonehash(lower(spellfix1_translit(X)))
> 
> ```

然后，这个关键字被限制在 "scope" 字符内。默认的范围值是 4，但可以在 WHERE 子句中使用 "scope=N" 术语指定替代范围。在关键字被截断之后，编辑距离针对词汇表中每个具有以简写键开头的 k2 值运行。

例如，假设输入单词是 "Paskagula"。音标键是 "BACACALA"，然后被截断为 4 个字符 "BACA"。然后对词汇表中具有以 BACA 开头的 k2 值的 4980 个条目（总共 272,597 个条目）运行编辑距离，得到 "Pascagoula" 作为最佳匹配。

只搜索具有匹配 langid 的词汇表条目。因此，同一表可以包含来自多种语言的条目，只会使用请求的语言。默认的 langid 是 0。

# 5\. 可配置的编辑距离

使用固定权重的内置瓦格纳编辑距离函数可以通过在创建虚拟表时指定 "edit_cost_table=*TABLENAME*" 参数到 spellfix1 模块来替换具有应用定义权重和支持 Unicode 的 editdist3() 编辑距离函数。例如：

> ```sql
> CREATE VIRTUAL TABLE demo2 USING spellfix1(edit_cost_table=APPCOST);
> 
> ```

editdist3() 编辑距离函数也可以通过将适当的字符串插入到虚拟表的 "command" 列来在运行时选择或取消选择：

> ```sql
> INSERT INTO demo2(command) VALUES('edit_cost_table=APPCOST');
> 
> ```

在上述示例中，将会查询 APPCOST 表以查找编辑距离系数。是 "edit_cost_table=" 参数在 spellfix1 模块名中的存在导致使用 editdist3() 代替内置的编辑距离函数。如果 APPCOST 是空字符串，则使用内置的瓦格纳编辑距离函数。

编辑距离系数通常从 APPCOST 表读取一次，然后存储在内存中。因此，对 APPCOST 表的运行时更改通常不会影响编辑距离结果。然而，将特殊字符串 'reset' 插入到虚拟表的 "command" 列会导致重新读取 APPCOST 表中的编辑距离系数。因此，当 APPCOST 表发生更改时，应用程序应运行类似以下的 SQL 语句：

> INSERT INTO demo2(command) VALUES("reset");

# 6\. 处理不寻常和困难的拼写

上述算法对大多数情况都很有效，但存在例外情况。这些例外情况可以通过在虚拟表中使用 "soundslike" 列添加额外条目来处理。

例如，许多源自希腊的单词以 "ps" 开头，其中 "p" 是无声的。例如：诗篇，假名，牛皮癣，心灵。另一个例子是，许多苏格兰姓氏可以以 "Mac" 或 "Mc" 开头拼写。因此，"MacKay" 和 "McKay" 的发音相同。

可以通过在虚拟表中为同一个单词添加其他拼写，并在 "soundslike" 列中添加备选拼写，来适应非按照其发音拼写的单词。例如，"诗篇" 的规范条目如下：

> ```sql
>   INSERT INTO demo(word) VALUES('psalm');
> 
> ```

为了增强将 "salm" 的拼写改正为 "psalm" 的能力，可以像这样添加一个条目：

> ```sql
>   INSERT INTO demo(word,soundslike) VALUES('psalm','salm');
> 
> ```

如果未指定 soundslike 值，则可以对同一个单词进行多个条目。请注意，如果未指定 soundslike 值，则 soundslike 默认为单词本身。

下面列出了一些可能需要添加额外 soundslike 条目的情况。具体条目将取决于应用程序和目标语言。

+   "ps" 开头的单词中的无声 "p"：诗篇，心灵

+   "pn" 开头的单词中的无声 "p"：肺炎，气动的

+   "pt" 开头的单词中的无声 "p"：翼龙，托勒密的

+   "dj" 开头的单词中的无声 "d"：精灵，雅加达

+   "kn" 开头的单词中的无声 "k"：骑士，Knuthson

+   单词开头带有"gn"的情况下的静音"g"：gnarly, gnome, gnat

+   苏格兰姓氏开头的"Mac"与"Mc"的区别

+   斯拉夫语单词中的"Tch"音：Tchaikovsky vs. Chaykovsky

+   西班牙语中"j"发音像"h"：LaJolla

+   以"wr"开头的单词与以"r"开头的单词的区别：write vs. rite

+   各种问题单词，如"debt"，"tsetse"，"Nguyen"，"Van Nuyes"。

# 7\. 辅助函数

实现拼写修复虚拟表的源代码模块还实现了几个 SQL 函数，这些函数对于使用 spellfix1 的应用程序或在开发使用 spellfix1 的应用程序时进行测试或诊断工作可能很有用。以下是可用的辅助函数：

**editdist3(P,W)**

editdist3(P,W,L)

editdist3(T)**

这些例程直接提供对 Wagner 编辑距离函数版本的访问，该函数允许应用程序定义编辑操作的权重。此函数的前两种形式比较模式 P 与单词 W 并返回编辑距离。在第一个函数中，langid 假定为 0，在第二个函数中，langid 由参数 L 给出。此函数的第三种形式从名为 T 的表重新加载编辑距离系数。

**spellfix1_editdist(P,W)**

此例程提供访问内置的 Wagner 编辑距离函数，该函数使用默认的固定成本。返回的值是将 W 转换为 P 所需的编辑距离。

**spellfix1_phonehash(X)**

此例程构建输入纯 ASCII 单词 X 的音素哈希并返回该哈希。此例程在 spellfix1 内部使用，以将阴影表的 K1 列转换为 K2 列。

**spellfix1_scriptcode(X)**

给定输入字符串 X，此例程尝试确定该输入的主要书写系统并返回该系统的 ISO-15924 数值代码。当前实现理解以下书写系统：

+   215 - Latin

+   220 - Cyrillic

+   200 - Greek

可能会在未来版本中添加更多语言代码。

**spellfix1_translit(X)**

此例程将 Unicode 文本转换为纯 ASCII，并返回输入文本 X 的纯 ASCII 表示。这是用于将词汇词转换为阴影表的 K1 列的内部使用函数。

# 8\. editdist3 函数

editdist3 算法是计算两个输入字符串之间最小编辑距离（也称为 Levenshtein 距离）的函数。editdist3 算法是 spellfix1 默认编辑距离函数的可配置替代方案。editdist3 的特点包括：

+   它适用于 Unicode（UTF8）文本。

+   应用程序可以提供插入、删除和替换成本的表。

+   在成本表中可以列举多字符的插入、删除和替换。

# 9\. editdist3 成本表

要编程 editdist3 的成本，请创建如下表：

> ```sql
> CREATE TABLE editcost(
>   iLang INT,   -- The language ID
>   cFrom TEXT,  -- Convert text from this
>   cTo   TEXT,  -- Convert text into this
>   iCost INT    -- The cost of doing the conversion
> );
> 
> ```

成本表可以任意命名 - 不一定要称为 "editcost"。表格可以包含额外的列。唯一的要求是表格必须包含上述四列，且列名必须与示例完全相同。

列 iLang 是一个非负整数，用于标识适用于特定语言的成本集合。函数 editdist3 在任何给定的编辑距离计算中只会使用单个 iLang 值。默认值是 0。建议只需要使用单一语言的应用始终对所有条目使用 iLang==0。

列 iCost 是将 cFrom 转换为 cTo 的数值成本。此值应为非负整数，且可能小于 100。默认单字符插入和删除成本为 100，单字符到单字符替换成本为 150。10000 或更高的成本被视为“无限”，并导致规则被忽略。

列 cFrom 和 cTo 显示编辑转换字符串。任一列或两列都可能包含多个字符。当 cFrom 为空时，表示插入 cTo 的成本。当 cTo 为空时，表示删除 cFrom 的成本。

在 spellfix1 算法中，cFrom 是用户输入的文本，cTo 是数据库中存在的正确拼写文本。editdist3 算法的目标是确定用户输入的文本与词典文本的接近程度。

成本表中有三个特例条目：

| cFrom | cTo | 意义 |
| --- | --- | --- |
| '' | '?' | 默认插入成本 |
| '?' | '' | 默认删除成本 |
| '?' | '?' | 默认替换成本 |

如果省略了任何上述特殊情况条目，则插入和删除的值将为 100，替换的值将为 150。要禁用默认插入、删除和/或替换，请将它们的相应成本设置为 10000 或更高。

成本表中的其他条目专门针对特定字符的转换。特定转换的成本应低于默认成本，否则默认成本将优先，而特定转换将永远不会被使用。

一些示例，成本表条目：

> ```sql
> INSERT INTO editcost(iLang, cFrom, cTo, iCost)
> VALUES(0, 'a', 'ä', 5);
> 
> ```

上述规则表明，用户输入的字母 "a" 可以与字典中的字母 "ä" 匹配，并罚分 5。

> ```sql
> INSERT INTO editcost(iLang, cFrom, cTo, iCost)
> VALUES(0, 'ss', 'ß', 8);
> 
> ```

cFrom 和 cTo 中的字符数不需要相同。上述规则表明，用户输入的 "ss" 将与字典中的 "ß" 匹配，并罚分 8。

# 10\. 尝试 editcost3() 函数

如果在创建 spellfix1 虚拟表时指定了 "edit_cost_table=TABLE" 选项作为参数，则 spellfix1 虚拟表将使用 editdist3。但是可以直接使用内置的 "editdist3()" SQL 函数来测试 editdist3。editdist3() SQL 函数有 3 种形式：

1.  editdist3('TABLENAME');

1.  editdist3('string1', 'string2');

1.  editdist3('string1', 'string2', langid);

第一种形式从名为 `'TABLENAME'` 的表中加载编辑距离系数。任何先前的系数都将被丢弃。因此，在尝试不同权重和权重表更改时，只需重新运行 `editdist3()` 的单参数形式即可重新加载修订后的系数。请注意，`editdist3()` SQL 函数使用的编辑距离权重与 `spellfix1` 虚拟表使用的权重是独立的。

第二种和第三种形式返回字符串 `'string1'` 和 `"string2'` 之间计算得到的编辑距离。在第二种形式中，使用语言标识符 0。语言标识符在第三种形式中指定。
