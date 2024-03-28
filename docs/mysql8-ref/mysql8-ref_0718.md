# 12.13 添加字符集

> 原文：[`dev.mysql.com/doc/refman/8.0/en/adding-character-set.html`](https://dev.mysql.com/doc/refman/8.0/en/adding-character-set.html)

12.13.1 字符定义数组

12.13.2 复杂字符集的字符串排序支持

12.13.3 复杂字符集的多字节字符支持

本节讨论了向 MySQL 添加字符集的过程。正确的过程取决于字符集是简单还是复杂：

+   如果字符集不需要用于排序的特殊字符串排序例程，也不需要多字节字符支持，则它是简单的。

+   如果字符集需要这些功能中的任何一个，它就是复杂的。

例如，`greek`和`swe7`是简单字符集，而`big5`和`czech`是复杂字符集。

要使用以下说明，您必须具有 MySQL 源代码分发。在说明中，*`MYSET`*代表您想要添加的字符集的名称。

1.  向`sql/share/charsets/Index.xml`文件的*`MYSET`*添加一个`<charset>`元素。使用文件中现有内容作为添加新内容的指南。`latin1` `<charset>`元素的部分列表如下：

    ```sql
    <charset name="latin1">
      <family>Western</family>
      <description>cp1252 West European</description>
      ...
      <collation name="latin1_swedish_ci" id="8" order="Finnish, Swedish">
        <flag>primary</flag>
        <flag>compiled</flag>
      </collation>
      <collation name="latin1_danish_ci" id="15" order="Danish"/>
      ...
      <collation name="latin1_bin" id="47" order="Binary">
        <flag>binary</flag>
        <flag>compiled</flag>
      </collation>
      ...
    </charset>
    ```

    `<charset>`元素必须列出字符集的所有排序规则。这些排序规则必须至少包括一个二进制排序规则和一个默认（主要）排序规则。默认排序规则通常使用`general_ci`（通用，不区分大小写）后缀命名。二进制排序规则可能是默认排序规则，但通常它们是不同的。默认排序规则应该有一个`primary`标志。二进制排序规则应该有一个`binary`标志。

    您必须为每个排序规则分配一个唯一的 ID 号。ID 范围从 1024 到 2047 保留用于用户定义的排序规则。要找到当前使用的排序规则 ID 的最大值，请使用此查询：

    ```sql
    SELECT MAX(ID) FROM INFORMATION_SCHEMA.COLLATIONS;
    ```

1.  此步骤取决于您是添加简单字符集还是复杂字符集。简单字符集只需要一个配置文件，而复杂字符集需要定义排序函数、多字节函数或两者的 C 源文件。

    对于简单字符集，创建一个配置文件，`*`MYSET`*.xml`，描述字符集属性。将此文件创建在`sql/share/charsets`目录中。您可以使用`latin1.xml`的副本作为此文件的基础。文件的语法非常简单：

    +   注释写作普通的 XML 注释（`<!-- *`text`* -->`）。

    +   `<map>`数组元素内的单词由任意数量的空格分隔。

    +   `<map>`数组元素内的每个单词必须是十六进制格式的数字。

    +   `<ctype>`元素的`<map>`数组元素有 257 个单词。之后的其他`<map>`数组元素有 256 个单词。参见第 12.13.1 节，“字符定义数组”。

    +   对于`Index.xml`中字符集的`<charset>`元素中列出的每个排序，`*`MYSET`*.xml`必须包含定义字符排序的`<collation>`元素。

    对于复杂字符集，创建一个描述字符集属性并定义支持操作所需的支持例程的 C 源文件：

    +   在`strings`目录中创建文件`ctype-*`MYSET`*.c`。查看现有的`ctype-*.c`文件（如`ctype-big5.c`）以了解需要定义的内容。您的文件中的数组必须具有类似`ctype_*`MYSET`*`、`to_lower_*`MYSET`*`等的名称。这些对应于简单字符集的数组。参见第 12.13.1 节，“字符定义数组”。

    +   对于`Index.xml`中字符集的`<charset>`元素中列出的每个`<collation>`元素，`ctype-*`MYSET`*.c`文件必须提供排序的实现。

    +   如果字符集需要字符串排序函数，请参见第 12.13.2 节，“复杂字符集的字符串排序支持”。

    +   如果字符集需要多字节字符支持，请参见第 12.13.3 节，“复杂字符集的多字节字符支持”。

1.  修改配置信息。使用现有的配置信息作为为*`MYSYS`*添加信息的指南。此示例假定字符集具有默认和二进制排序，但如果*`MYSET`*具有其他排序，则需要更多行。

    1.  编辑`mysys/charset-def.c`，并“注册”新字符集的排序。

        将这些行添加到“声明”部分：

        ```sql
        #ifdef HAVE_CHARSET_*MYSET*
        extern CHARSET_INFO my_charset_*MYSET*_general_ci;
        extern CHARSET_INFO my_charset_*MYSET*_bin;
        #endif
        ```

        将这些行添加到“注册”部分：

        ```sql
        #ifdef HAVE_CHARSET_*MYSET*
          add_compiled_collation(&my_charset_*MYSET*_general_ci);
          add_compiled_collation(&my_charset_*MYSET*_bin);
        #endif
        ```

    1.  如果字符集使用`ctype-*`MYSET`*.c`，编辑`strings/CMakeLists.txt`并将`ctype-*`MYSET`*.c`添加到`STRINGS_SOURCES`变量的定义中。

    1.  编辑`cmake/character_sets.cmake`：

        1.  将*`MYSET`*添加到`CHARSETS_AVAILABLE`的值中，按字母顺序排列。

        1.  将*`MYSET`*按字母顺序添加到`CHARSETS_COMPLEX`的值中。即使对于简单字符集也需要这样做，以便**CMake**可以识别`-DDEFAULT_CHARSET=*`MYSET`*`。

1.  重新配置、重新编译并测试。
