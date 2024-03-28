> 原文：[`dev.mysql.com/doc/refman/8.0/en/ldml-rules.html`](https://dev.mysql.com/doc/refman/8.0/en/ldml-rules.html)

#### 12.14.4.2 MySQL 支持的 LDML 语法

本节描述了 MySQL 识别的 LDML 语法。这是 LDML 规范中描述的语法的子集，可在[`www.unicode.org/reports/tr35/`](http://www.unicode.org/reports/tr35/)找到，应该查阅以获取更多信息。MySQL 识别了足够大的语法子集，因此在许多情况下，可以从 Unicode 通用区域数据存储库下载排序定义，并将相关部分（即在`<rules>`和`</rules>`标记之间的部分）粘贴到 MySQL 的`Index.xml`文件中。这里描述的规则都受支持，只是字符排序仅在主要级别发生。指定在次要或更高排序级别上存在差异的规则被识别（因此可以包含在排序定义中），但在主要级别被视为相等。

当 MySQL 服务器在解析`Index.xml`文件时发现问题时，会生成诊断信息。请参阅第 12.14.4.3 节，“解析 Index.xml 时的诊断”。

**字符表示**

在 LDML 规则中命名的字符可以直接写出，也可以用`\u*`nnnn`*`格式，其中*`nnnn`*是十六进制 Unicode 代码点值。例如，`A`和`á`可以直接写出，也可以写为`\u0041`和`\u00E1`。在十六进制值中，字母`A`到`F`不区分大小写；`\u00E1`和`\u00e1`是等效的。对于 UCA 4.0.0 排序，只能对基本多文种平面中的字符使用十六进制表示法，而不能对超出`0000`到`FFFF`的 BMP 范围的字符使用。对于 UCA 5.2.0 排序，可以对任何字符使用十六进制表示法。

`Index.xml`文件本身应使用 UTF-8 编码编写。

**语法规则**

LDML 具有重置规则和移位规则，用于指定字符排序。排序规则被给定为一组以建立锚点的重置规则开头的规则，然后是指示字符相对于锚点如何排序的移位规则。

+   `<reset>`规则本身不指定任何排序。相反，它“重置”后续移位规则，使它们相对于给定字符进行排序。以下任一规则都会将后续移位规则重置为相对于字母`'A'`进行排序：

    ```sql
    <reset>A</reset>

    <reset>\u0041</reset>
    ```

+   `<p>`、`<s>`和`<t>`移位规则定义了一个字符与另一个字符的主要、次要和三级差异：

    +   使用主要差异来区分不同的字母。

    +   使用次要差异来区分重音变体。

    +   使用三级差异来区分大小写变体。

    以下任一规则指定了字符`'G'`的主要移位规则：

    ```sql
    <p>G</p>

    <p>\u0047</p>
    ```

+   `<i>`移位规则表示一个字符与另一个字符排序相同。以下规则使`'b'`与`'a'`排序相同：

    ```sql
    <reset>a</reset>
    <i>b</i>
    ```

+   缩写的移位语法使用一对标签指定多个移位规则。以下表格显示了缩写语法规则与等效的非缩写规则之间的对应关系。

    **表 12.5 缩写的移位语法**

    | 缩写语法 | 非缩写语法 |
    | --- | --- |
    | `<pc>xyz</pc>` | `<p>x</p><p>y</p><p>z</p>` |
    | `<sc>xyz</sc>` | `<s>x</s><s>y</s><s>z</s>` |
    | `<tc>xyz</tc>` | `<t>x</t><t>y</t><t>z</t>` |
    | `<ic>xyz</ic>` | `<i>x</i><i>y</i><i>z</i>` |

+   扩展是一个重置规则，为多字符序列建立锚点。MySQL 支持 2 到 6 个字符长的扩展。以下规则将`'z'`放在主级别比三个字符序列`'abc'`更大：

    ```sql
    <reset>abc</reset>
    <p>z</p>
    ```

+   缩写是将多字符序列排序的移位规则。MySQL 支持 2 到 6 个字符长的缩写。以下规则将三个字符序列`'xyz'`放在主级别比`'a'`更大：

    ```sql
    <reset>a</reset>
    <p>xyz</p>
    ```

+   长扩展和长缩写可以一起使用。这些规则将三个字符序列`'xyz'`放在主级别比三个字符序列`'abc'`更大：

    ```sql
    <reset>abc</reset>
    <p>xyz</p>
    ```

+   正常的扩展语法使用`<x>`加上`<extend>`元素来指定一个扩展。以下规则将字符`'k'`放在二级别比序列`'ch'`更大。也就是说，`'k'`的行为就像它扩展到`'c'`后跟着`'h'`的字符一样：

    ```sql
    <reset>c</reset>
    <x><s>k</s><extend>h</extend></x>
    ```

    这种语法允许长序列。这些规则将序列`'ccs'`放在三级别比序列`'cscs'`更大：

    ```sql
    <reset>cs</reset>
    <x><t>ccs</t><extend>cs</extend></x>
    ```

    LDML 规范将正常扩展语法描述为“棘手”。详细信息请参阅该规范。

+   先前的上下文语法使用`<x>`加上`<context>`元素来指定字符前的上下文影响其排序方式。以下规则将`'-'`放在二级比`'a'`更大，但仅当`'-'`出现在`'b'`之后时：

    ```sql
    <reset>a</reset>
    <x><context>b</context><s>-</s></x>
    ```

+   先前的上下文语法可以包括`<extend>`元素。这些规则将`'def'`放在主级别比`'aghi'`更大，但仅当`'def'`出现在`'abc'`之后时：

    ```sql
    <reset>a</reset>
    <x><context>abc</context><p>def</p><extend>ghi</extend></x>
    ```

+   重置规则允许`before`属性。通常，在重置规则之后的移位规则指示在重置字符之后排序的字符。在具有`before`属性的重置规则之后的移位规则指示在重置字符之前排序的字符。以下规则将字符`'b'`立即放在主级别的`'a'`之前：

    ```sql
    <reset before="primary">a</reset>
    <p>b</p>
    ```

    允许的`before`属性值通过名称或等效的数值指定排序级别：

    ```sql
    <reset before="primary">
    <reset before="1">

    <reset before="secondary">
    <reset before="2">

    <reset before="tertiary">
    <reset before="3">
    ```

+   重置规则可以命名逻辑重置位置而不是文字字符：

    ```sql
    <first_tertiary_ignorable/>
    <last_tertiary_ignorable/>
    <first_secondary_ignorable/>
    <last_secondary_ignorable/>
    <first_primary_ignorable/>
    <last_primary_ignorable/>
    <first_variable/>
    <last_variable/>
    <first_non_ignorable/>
    <last_non_ignorable/>
    <first_trailing/>
    <last_trailing/>
    ```

    这些规则将`'z'`放在主级别比具有默认 Unicode 排序元素表（DUCET）条目且不是 CJK 的不可忽略字符更大：

    ```sql
    <reset><last_non_ignorable/></reset>
    <p>z</p>
    ```

    逻辑位置在下表中显示了代码点。

    **表 12.6 逻辑重置位置代码点**

    | 逻辑位置 | Unicode 4.0.0 代码点 | Unicode 5.2.0 代码点 |
    | --- | --- | --- |
    | `<first_non_ignorable/>` | U+02D0 | U+02D0 |
    | `<last_non_ignorable/>` | U+A48C | U+1342E |
    | `<first_primary_ignorable/>` | U+0332 | U+0332 |
    | `<last_primary_ignorable/>` | U+20EA | U+101FD |
    | `<first_secondary_ignorable/>` | U+0000 | U+0000 |
    | `<last_secondary_ignorable/>` | U+FE73 | U+FE73 |
    | `<first_tertiary_ignorable/>` | U+0000 | U+0000 |
    | `<last_tertiary_ignorable/>` | U+FE73 | U+FE73 |
    | `<first_trailing/>` | U+0000 | U+0000 |
    | `<last_trailing/>` | U+0000 | U+0000 |
    | `<first_variable/>` | U+0009 | U+0009 |
    | `<last_variable/>` | U+2183 | U+1D371 |
    | 逻辑位置 | Unicode 4.0.0 代码点 | Unicode 5.2.0 代码点 |

+   `<collation>`元素允许`shift-after-method`属性，该属性影响移位规则的字符权重计算。该属性有以下允许的值：

    +   `simple`: 计算字符权重，就像没有`before`属性的重置规则一样。如果未给出属性，则这是默认值。

    +   `expand`: 使用扩展来处理重置规则后的偏移。

    假设`'0'`和`'1'`的权重分别为`0E29`和`0E2A`，我们想在`'0'`和`'1'`之间放置所有基本拉丁字母：

    ```sql
    <reset>0</reset>
    <pc>abcdefghijklmnopqrstuvwxyz</pc>
    ```

    对于简单的移位模式，权重计算如下：

    ```sql
    'a' has weight 0E29+1
    'b' has weight 0E29+2
    'c' has weight 0E29+3
    ...
    ```

    然而，在`'0'`和`'1'`之间没有足够的空位来放置 26 个字符。结果是数字和字母混合在一起。

    要解决这个问题，使用`shift-after-method="expand"`。然后权重计算如下：

    ```sql
    'a' has weight [0E29][233D+1]
    'b' has weight [0E29][233D+2]
    'c' has weight [0E29][233D+3]
    ...
    ```

    `233D` 是字符`0xA48C`的 UCA 4.0.0 权重，它是最后一个非可忽略字符（排序中最大的字符，不包括 CJK）。UCA 5.2.0 类似，但使用`3ACA`，对于字符`0x1342E`。

**MySQL 特定的 LDML 扩展**

LDML 规则的扩展允许`<collation>`元素在`<collation>`标签中包含一个可选的`version`属性，以指示排序所基于的 UCA 版本。如果省略`version`属性，则其默认值为`4.0.0`。例如，此规范指示基于 UCA 5.2.0 的排序：

```sql
<collation id="*nnn*" name="utf8mb4_*xxx*_ci" version="5.2.0">
...
</collation>
```
