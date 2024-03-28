# 14.9.9 MeCab 全文解析器插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/fulltext-search-mecab.html`](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search-mecab.html)

MySQL 内置的全文解析器使用单词之间的空格作为分隔符来确定单词的起始和结束位置，这在处理不使用单词分隔符的表意语言时存在限制。为了解决这个问题，MySQL 为日语提供了 MeCab 全文解析器插件。MeCab 全文解析器插件支持与`InnoDB`和`MyISAM`一起使用。

注意

MySQL 还提供了支持日语的 ngram 全文解析器插件。有关更多信息，请参见第 14.9.8 节“ngram 全文解析器”。

MeCab 全文解析器插件是用于日语的全文解析器插件，将文本序列标记为有意义的单词。例如，MeCab 将“データベース管理”（“数据库管理”）标记为“データベース”（“数据库”）和“管理”（“管理”）。相比之下，ngram 全文解析器将文本标记为连续的*`n`*个字符序列，其中*`n`*表示 1 到 10 之间的数字。

除了将文本标记为有意义的单词外，MeCab 索引通常比 ngram 索引小，并且 MeCab 全文搜索通常更快。一个缺点是与 ngram 全文解析器相比，MeCab 全文解析器可能需要更长的时间来标记文档。

第 14.9 节“全文搜索函数”中描述的全文搜索语法适用于 MeCab 解析器插件。本节描述了解析行为的差异。全文搜索相关的配置选项也适用。

有关 MeCab 解析器的更多信息，请参考 Github 上的[MeCab: Yet Another Part-of-Speech and Morphological Analyzer](http://taku910.github.io/mecab/)项目。

#### 安装 MeCab 解析器插件

MeCab 解析器插件需要`mecab`和`mecab-ipadic`。

在支持的 Fedora、Debian 和 Ubuntu 平台上（除了 Ubuntu 12.04，系统中的`mecab`版本太旧），如果`mecab`安装在默认位置，则 MySQL 会动态链接到系统的`mecab`安装。在其他支持的类 Unix 平台上，`libmecab.so`静态链接在`libpluginmecab.so`中，该文件位于 MySQL 插件目录中。`mecab-ipadic`包含在 MySQL 二进制文件中，位于`*`MYSQL_HOME`*\lib\mecab`中。

你可以使用本机包管理工具（在 Fedora、Debian 和 Ubuntu 上）安装`mecab`和`mecab-ipadic`，也可以从源代码构建`mecab`和`mecab-ipadic`。有关使用本机包管理工具安装`mecab`和`mecab-ipadic`的信息，请参见从二进制发行版安装 MeCab（可选）。如果你想从源代码构建`mecab`和`mecab-ipadic`，请参见从源代码构建 MeCab（可选）。

在 Windows 上，`libmecab.dll`位于 MySQL 的`bin`目录中。`mecab-ipadic`位于`*`MYSQL_HOME`*/lib/mecab`中。

要安装和配置 MeCab 解析器插件，请执行以下步骤：

1.  在 MySQL 配置文件中，将`mecab_rc_file`配置选项设置为`mecabrc`配置文件的位置，该文件是 MeCab 的配置文件。如果你使用 MySQL 分发的 MeCab 包，`mecabrc` 文件位于`MYSQL_HOME/lib/mecab/etc/`。

    ```sql
    [mysqld]
    loose-mecab-rc-file=MYSQL_HOME/lib/mecab/etc/mecabrc
    ```

    `loose`前缀是一个选项修饰符。在安装 MeCab 解析器插件之前，MySQL 不会识别`mecab_rc_file`选项，但必须在尝试安装 MeCab 解析器插件之前设置它。`loose`前缀允许你重新启动 MySQL，而不会因为无法识别的变量而遇到错误。

    如果你使用自己的 MeCab 安装，或者从源代码构建 MeCab，`mecabrc`配置文件的位置可能会有所不同。

    有关 MySQL 配置文件及其位置的信息，请参见 Section 6.2.2.2，“使用选项文件”。

1.  同样在 MySQL 配置文件中，将最小标记大小设置为 1 或 2，这是与 MeCab 解析器一起使用时推荐的值。对于`InnoDB`表，最小标记大小由`innodb_ft_min_token_size`配置选项定义，默认值为 3。对于`MyISAM`表，最小标记大小由`ft_min_word_len`定义，默认值为 4。

    ```sql
    [mysqld]
    innodb_ft_min_token_size=1
    ```

1.  修改`mecabrc`配置文件以指定你想使用的字典。MySQL 二进制文件中分发的`mecab-ipadic`包含三个字典（`ipadic_euc-jp`、`ipadic_sjis`和`ipadic_utf-8`）。MySQL 打包的`mecabrc`配置文件包含类似以下条目：

    ```sql
    dicdir =  /path/to/mysql/lib/mecab/lib/mecab/dic/ipadic_euc-jp
    ```

    要使用`ipadic_utf-8`字典，例如，修改条目如下：

    ```sql
    dicdir=*MYSQL_HOME*/lib/mecab/dic/ipadic_utf-8
    ```

    如果您正在使用自己的 MeCab 安装或已经从源代码构建了 MeCab，那么`mecabrc`文件中的默认`dicdir`条目可能会有所不同，字典及其位置也会不同。

    注意

    安装完 MeCab 解析器插件后，您可以使用`mecab_charset`状态变量查看与 MeCab 一起使用的字符集。MySQL 二进制文件提供的三个 MeCab 字典支持以下字符集。

    +   `ipadic_euc-jp`字典支持`ujis`和`eucjpms`字符集。

    +   `ipadic_sjis`字典支持`sjis`和`cp932`字符集。

    +   `ipadic_utf-8`字典支持`utf8mb3`和`utf8mb4`字符集。

    `mecab_charset`仅报告第一个支持的字符集。例如，`ipadic_utf-8`字典支持`utf8mb3`和`utf8mb4`。当使用此字典时，`mecab_charset`总是报告`utf8`。

1.  重新启动 MySQL。

1.  安装 MeCab 解析器插件：

    使用`INSTALL PLUGIN`安装 MeCab 解析器插件。插件名称为`mecab`，共享库名称为`libpluginmecab.so`。有关安装插件的其他信息，请参见 Section 7.6.1, “Installing and Uninstalling Plugins”。

    ```sql
    INSTALL PLUGIN mecab SONAME 'libpluginmecab.so';
    ```

    安装完成后，MeCab 解析器插件会在每次正常 MySQL 重启时加载。

1.  使用`SHOW PLUGINS`语句验证 MeCab 解析器插件是否已加载。

    ```sql
    mysql> SHOW PLUGINS;
    ```

    `mecab`插件应该出现在插件列表中。

#### 创建使用 MeCab 解析器的 FULLTEXT 索引

要创建一个使用 mecab 解析器的`FULLTEXT`索引，请在`CREATE TABLE`、`ALTER TABLE`或`CREATE INDEX`中指定`WITH PARSER ngram`。

此示例演示了创建带有`mecab` `FULLTEXT`索引的表，插入示例数据，并在 Information Schema `INNODB_FT_INDEX_CACHE`表中查看标记化数据：

```sql
mysql> USE test;

mysql> CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT,
      FULLTEXT (title,body) WITH PARSER mecab
    ) ENGINE=InnoDB CHARACTER SET utf8mb4;

mysql> SET NAMES utf8mb4;

mysql> INSERT INTO articles (title,body) VALUES
    ('データベース管理','このチュートリアルでは、私はどのようにデータベースを管理する方法を紹介します'),
    ('データベースアプリケーション開発','データベースアプリケーションを開発することを学ぶ');

mysql> SET GLOBAL innodb_ft_aux_table="test/articles";

mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE ORDER BY doc_id, position;
```

要向现有表添加`FULLTEXT`索引，可以使用`ALTER TABLE`或`CREATE INDEX`。例如：

```sql
CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT
     ) ENGINE=InnoDB CHARACTER SET utf8mb4;

ALTER TABLE articles ADD FULLTEXT INDEX ft_index (title,body) WITH PARSER mecab;

# Or:

CREATE FULLTEXT INDEX ft_index ON articles (title,body) WITH PARSER mecab;
```

#### MeCab 解析器空格处理

MeCab 解析器在查询字符串中使用空格作为分隔符。例如，MeCab 解析器将データベース管理标记为データベース和管理。

#### MeCab 解析器停用词处理

默认情况下，MeCab 解析器使用默认的停用词列表，其中包含一小部分英文停用词。要使用适用于日语的停用词列表，您必须创建自己的停用词列表。有关创建停用词列表的信息，请参见 第 14.9.4 节，“全文停用词”。

#### MeCab 解析器术语搜索

对于自然语言模式搜索，搜索词被转换为标记的并集。例如，データベース管理 被转换为 データベース 管理。

```sql
SELECT COUNT(*) FROM articles WHERE MATCH(title,body) AGAINST('データベース管理' IN NATURAL LANGUAGE MODE);
```

对于布尔模式搜索，搜索词被转换为搜索短语。例如，データベース管理 被转换为 データベース 管理。

```sql
SELECT COUNT(*) FROM articles WHERE MATCH(title,body) AGAINST('データベース管理' IN BOOLEAN MODE);
```

#### MeCab 解析器通配符搜索

通配符搜索词不被标记。对前缀 データベース管理* 进行搜索时，会在前缀 データベース管理 上执行搜索。

```sql
SELECT COUNT(*) FROM articles WHERE MATCH(title,body) AGAINST('データベース*' IN BOOLEAN MODE);
```

#### MeCab 解析器短语搜索

短语被标记。例如，データベース管理 被标记为 データベース 管理。

```sql
SELECT COUNT(*) FROM articles WHERE MATCH(title,body) AGAINST('"データベース管理"' IN BOOLEAN MODE);
```

#### 从二进制发行版安装 MeCab（可选）

本节描述了如何使用本机软件包管理工具从二进制发行版安装 `mecab` 和 `mecab-ipadic`。例如，在 Fedora 上，您可以使用 Yum 执行安装：

```sql
yum mecab-devel
```

在 Debian 或 Ubuntu 上，您可以执行 APT 安装：

```sql
apt-get install mecab
apt-get install mecab-ipadic
```

#### 从源代码安装 MeCab（可选）

如果您想要从源代码构建 `mecab` 和 `mecab-ipadic`，以下是基本的安装步骤。有关更多信息，请参考 MeCab 文档。

1.  从 [`taku910.github.io/mecab/#download`](http://taku910.github.io/mecab/#download) 下载 `mecab` 和 `mecab-ipadic` 的 tar.gz 软件包。截至 2016 年 2 月，最新可用的软件包是 `mecab-0.996.tar.gz` 和 `mecab-ipadic-2.7.0-20070801.tar.gz`。

1.  安装 `mecab`：

    ```sql
    tar zxfv mecab-0.996.tar
    cd mecab-0.996
    ./configure
    make
    make check
    su
    make install
    ```

1.  安装 `mecab-ipadic`：

    ```sql
    tar zxfv mecab-ipadic-2.7.0-20070801.tar
    cd mecab-ipadic-2.7.0-20070801
    ./configure
    make
    su
    make install
    ```

1.  使用 `WITH_MECAB` CMake 选项编译 MySQL。如果您已将 `mecab` 和 `mecab-ipadic` 安装到默认位置，请将 `WITH_MECAB` 选项设置为 `system`。

    ```sql
    -DWITH_MECAB=system
    ```

    如果您定义了自定义安装目录，请将 `WITH_MECAB` 设置为自定义目录。例如：

    ```sql
    -DWITH_MECAB=/path/to/mecab
    ```
