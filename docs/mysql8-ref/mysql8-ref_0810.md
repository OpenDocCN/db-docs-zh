# 14.9.4 全文停用词

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-stopwords.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-stopwords.html)

停用词列表是使用服务器字符集和排序规则（`character_set_server`和`collation_server`系统变量的值）加载和搜索全文查询的。如果停用词文件或用于全文索引或搜索的列的字符集或排序规则与`character_set_server`或`collation_server`不同，停用词查找可能会出现错误的命中或遗漏。

停用词查找的大小写敏感性取决于服务器排序规则。例如，如果排序规则是`utf8mb4_0900_ai_ci`，则查找是不区分大小写的，而如果排序规则是`utf8mb4_0900_as_cs`或`utf8mb4_bin`，则查找是区分大小写的。

+   InnoDB 搜索索引的停用词

+   MyISAM 搜索索引的停用词

#### InnoDB 搜索索引的停用词

`InnoDB`具有相对较短的默认停用词列表，因为来自技术、文学和其他来源的文档通常使用短词作为关键词或重要短语。例如，您可能搜索“to be or not to be”并期望得到一个合理的结果，而不是忽略所有这些单词。

要查看默认的`InnoDB`停用词列表，请查询信息模式`INNODB_FT_DEFAULT_STOPWORD`表。

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
+-------+
| value |
+-------+
| a     |
| about |
| an    |
| are   |
| as    |
| at    |
| be    |
| by    |
| com   |
| de    |
| en    |
| for   |
| from  |
| how   |
| i     |
| in    |
| is    |
| it    |
| la    |
| of    |
| on    |
| or    |
| that  |
| the   |
| this  |
| to    |
| was   |
| what  |
| when  |
| where |
| who   |
| will  |
| with  |
| und   |
| the   |
| www   |
+-------+
36 rows in set (0.00 sec)
```

要为所有`InnoDB`表定义自己的停用词列表，请定义一个与`INNODB_FT_DEFAULT_STOPWORD`表结构相同的表，填充它的停用词，并在创建全文索引之前将`innodb_ft_server_stopword_table`选项的值设置为形式为`*`db_name`*/*`table_name`*`的值。停用词表必须有一个名为`value`的单个`VARCHAR`列。以下示例演示了为`InnoDB`创建和配置新的全局停用词表。

```sql
-- Create a new stopword table

mysql> CREATE TABLE my_stopwords(value VARCHAR(30)) ENGINE = INNODB;
Query OK, 0 rows affected (0.01 sec)

-- Insert stopwords (for simplicity, a single stopword is used in this example)

mysql> INSERT INTO my_stopwords(value) VALUES ('Ishmael');
Query OK, 1 row affected (0.00 sec)

-- Create the table

mysql> CREATE TABLE opening_lines (
id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
opening_line TEXT(500),
author VARCHAR(200),
title VARCHAR(200)
) ENGINE=InnoDB;
Query OK, 0 rows affected (0.01 sec)

-- Insert data into the table

mysql> INSERT INTO opening_lines(opening_line,author,title) VALUES
('Call me Ishmael.','Herman Melville','Moby-Dick'),
('A screaming comes across the sky.','Thomas Pynchon','Gravity\'s Rainbow'),
('I am an invisible man.','Ralph Ellison','Invisible Man'),
('Where now? Who now? When now?','Samuel Beckett','The Unnamable'),
('It was love at first sight.','Joseph Heller','Catch-22'),
('All this happened, more or less.','Kurt Vonnegut','Slaughterhouse-Five'),
('Mrs. Dalloway said she would buy the flowers herself.','Virginia Woolf','Mrs. Dalloway'),
('It was a pleasure to burn.','Ray Bradbury','Fahrenheit 451');
Query OK, 8 rows affected (0.00 sec)
Records: 8  Duplicates: 0  Warnings: 0

-- Set the innodb_ft_server_stopword_table option to the new stopword table

mysql> SET GLOBAL innodb_ft_server_stopword_table = 'test/my_stopwords';
Query OK, 0 rows affected (0.00 sec)

-- Create the full-text index (which rebuilds the table if no FTS_DOC_ID column is defined)

mysql> CREATE FULLTEXT INDEX idx ON opening_lines(opening_line);
Query OK, 0 rows affected, 1 warning (1.17 sec)
Records: 0  Duplicates: 0  Warnings: 1
```

通过查询信息模式`INNODB_FT_INDEX_TABLE`表来验证指定的停用词（'Ishmael'）是否出现。

注意

默认情况下，长度小于 3 个字符或大于 84 个字符的单词不会出现在`InnoDB`全文搜索索引中。最大和最小单词长度值可通过`innodb_ft_max_token_size`和`innodb_ft_min_token_size`变量进行配置。此默认行为不适用于 ngram 解析器插件。 ngram 标记大小由`ngram_token_size`选项定义。

```sql
mysql> SET GLOBAL innodb_ft_aux_table='test/opening_lines';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT word FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_TABLE LIMIT 15;
+-----------+
| word      |
+-----------+
| across    |
| all       |
| burn      |
| buy       |
| call      |
| comes     |
| dalloway  |
| first     |
| flowers   |
| happened  |
| herself   |
| invisible |
| less      |
| love      |
| man       |
+-----------+
15 rows in set (0.00 sec)
```

要逐表创建停用词列表，请创建其他停用词表，并在创建全文索引之前使用`innodb_ft_user_stopword_table`选项指定要使用的停用词表。

#### `MyISAM`搜索索引的停用词

如果`character_set_server`为`ucs2`、`utf16`、`utf16le`或`utf32`，则使用`latin1`加载和搜索停用词文件。

要覆盖`MyISAM`表的默认停用词列表，请设置`ft_stopword_file`系统变量。（参见第 7.1.8 节，“服务器系统变量”。）变量值应为包含停用词列表的文件的路径名，或空字符串以禁用停用词过滤。服务器会在数据目录中查找该文件，除非给定绝对路径名以指定不同目录。更改此变量的值或停用词文件的内容后，重新启动服务器并重建您的`FULLTEXT`索引。

停用词列表是自由格式的，用任何非字母数字字符（如换行符、空格或逗号）分隔停用词。下划线字符（`_`）和单引号（`'`）是单词的一部分。停用词列表的字符集是服务器的默认字符集；请参阅第 12.3.2 节，“服务器字符集和排序规则”。

以下列表显示了`MyISAM`搜索索引的默认停用词。在 MySQL 源分发中，您可以在`storage/myisam/ft_static.c`文件中找到此列表。

```sql
a's           able          about         above         according
accordingly   across        actually      after         afterwards
again         against       ain't         all           allow
allows        almost        alone         along         already
also          although      always        am            among
amongst       an            and           another       any
anybody       anyhow        anyone        anything      anyway
anyways       anywhere      apart         appear        appreciate
appropriate   are           aren't        around        as
aside         ask           asking        associated    at
available     away          awfully       be            became
because       become        becomes       becoming      been
before        beforehand    behind        being         believe
below         beside        besides       best          better
between       beyond        both          brief         but
by            c'mon         c's           came          can
can't         cannot        cant          cause         causes
certain       certainly     changes       clearly       co
com           come          comes         concerning    consequently
consider      considering   contain       containing    contains
corresponding could         couldn't      course        currently
definitely    described     despite       did           didn't
different     do            does          doesn't       doing
don't         done          down          downwards     during
each          edu           eg            eight         either
else          elsewhere     enough        entirely      especially
et            etc           even          ever          every
everybody     everyone      everything    everywhere    ex
exactly       example       except        far           few
fifth         first         five          followed      following
follows       for           former        formerly      forth
four          from          further       furthermore   get
gets          getting       given         gives         go
goes          going         gone          got           gotten
greetings     had           hadn't        happens       hardly
has           hasn't        have          haven't       having
he            he's          hello         help          hence
her           here          here's        hereafter     hereby
herein        hereupon      hers          herself       hi
him           himself       his           hither        hopefully
how           howbeit       however       i'd           i'll
i'm           i've          ie            if            ignored
immediate     in            inasmuch      inc           indeed
indicate      indicated     indicates     inner         insofar
instead       into          inward        is            isn't
it            it'd          it'll         it's          its
itself        just          keep          keeps         kept
know          known         knows         last          lately
later         latter        latterly      least         less
lest          let           let's         like          liked
likely        little        look          looking       looks
ltd           mainly        many          may           maybe
me            mean          meanwhile     merely        might
more          moreover      most          mostly        much
must          my            myself        name          namely
nd            near          nearly        necessary     need
needs         neither       never         nevertheless  new
next          nine          no            nobody        non
none          noone         nor           normally      not
nothing       novel         now           nowhere       obviously
of            off           often         oh            ok
okay          old           on            once          one
ones          only          onto          or            other
others        otherwise     ought         our           ours
ourselves     out           outside       over          overall
own           particular    particularly  per           perhaps
placed        please        plus          possible      presumably
probably      provides      que           quite         qv
rather        rd            re            really        reasonably
regarding     regardless    regards       relatively    respectively
right         said          same          saw           say
saying        says          second        secondly      see
seeing        seem          seemed        seeming       seems
seen          self          selves        sensible      sent
serious       seriously     seven         several       shall
she           should        shouldn't     since         six
so            some          somebody      somehow       someone
something     sometime      sometimes     somewhat      somewhere
soon          sorry         specified     specify       specifying
still         sub           such          sup           sure
t's           take          taken         tell          tends
th            than          thank         thanks        thanx
that          that's        thats         the           their
theirs        them          themselves    then          thence
there         there's       thereafter    thereby       therefore
therein       theres        thereupon     these         they
they'd        they'll       they're       they've       think
third         this          thorough      thoroughly    those
though        three         through       throughout    thru
thus          to            together      too           took
toward        towards       tried         tries         truly
try           trying        twice         two           un
under         unfortunately unless        unlikely      until
unto          up            upon          us            use
used          useful        uses          using         usually
value         various       very          via           viz
vs            want          wants         was           wasn't
way           we            we'd          we'll         we're
we've         welcome       well          went          were
weren't       what          what's        whatever      when
whence        whenever      where         where's       whereafter
whereas       whereby       wherein       whereupon     wherever
whether       which         while         whither       who
who's         whoever       whole         whom          whose
why           will          willing       wish          with
within        without       won't         wonder        would
wouldn't      yes           yet           you           you'd
you'll        you're        you've        your          yours
yourself      yourselves    zero
```
