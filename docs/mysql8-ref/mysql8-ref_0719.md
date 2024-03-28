# 12.13.1 字符定义数组

> [`dev.mysql.com/doc/refman/8.0/en/character-arrays.html`](https://dev.mysql.com/doc/refman/8.0/en/character-arrays.html)

每个简单字符集都有一个配置文件，位于 `sql/share/charsets` 目录中。 对于名为 *`MYSYS`* 的字符集，文件名为 `*`MYSET`*.xml`。 它使用 `<map>` 数组元素列出字符集属性。 `<map>` 元素出现在这些元素内部：

+   `<ctype>` 为每个字符定义属性。

+   `<lower>` 和 `<upper>` 列出小写和大写字符。

+   `<unicode>` 将 8 位字符值映射到 Unicode 值。

+   `<collation>` 元素指示用于比较和排序的字符排序，每个排序一个元素。 二进制排序不需要 `<map>` 元素，因为字符代码本身提供排序。

对于在 `strings` 目录中的 `ctype-*`MYSET`*.c` 文件中实现的复杂字符集，有相应的数组：`ctype_*`MYSET`*[]`，`to_lower_*`MYSET`*[]` 等等。 并非每个复杂字符集都有所有数组。 有关示例，请参见现有的 `ctype-*.c` 文件。 有关更多信息，请参见 `strings` 目录中的 `CHARSET_INFO.txt` 文件。

大多数数组按字符值索引，并具有 256 个元素。 `<ctype>` 数组按字符值 + 1 索引，并具有 257 个元素。 这是处理 `EOF` 的传统约定。

`<ctype>` 数组元素是位值。 每个元素描述字符集中单个字符的属性。 每个属性与位掩码关联，如 `include/m_ctype.h` 中定义的：

```sql
#define _MY_U   01 /* Upper case */
#define _MY_L   02 /* Lower case */
#define _MY_NMR 04 /* Numeral (digit) */
#define _MY_SPC 010 /* Spacing character */
#define _MY_PNT 020 /* Punctuation */
#define _MY_CTR 040 /* Control character */
#define _MY_B   0100 /* Blank */
#define _MY_X   0200 /* heXadecimal digit */
```

给定字符的 `<ctype>` 值应该是描述该字符的适用位掩码值的并集。 例如，`'A'` 是大写字符（`_MY_U`）以及十六进制数字（`_MY_X`），因此其 `ctype` 值应该定义如下：

```sql
ctype['A'+1] = _MY_U | _MY_X = 01 | 0200 = 0201
```

`m_ctype.h` 中的位掩码值是八进制值，但在 `*`MYSET`*.xml` 中的 `<ctype>` 数组的元素应该以十六进制值编写。

`<lower>` 和 `<upper>` 数组保存与字符集中每个成员对应的小写和大写字符。 例如：

```sql
lower['A'] should contain 'a'
upper['a'] should contain 'A'
```

每个 `<collation>` 数组指示字符应如何排序以进行比较和排序。 MySQL 根据此信息的值对字符进行排序。 在某些情况下，这与 `<upper>` 数组相同，这意味着排序不区分大小写。 对于更复杂的排序规则（用于复杂字符集），请参阅 12.13.2 “复杂字符集的字符串排序支持” 中有关字符串排序的讨论。
