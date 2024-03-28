# 12.8.6 排序规则效果示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-collation-effect.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-effect.html)

**示例 1：排序德语 Umlauts**

假设表`T`中的列`X`具有这些`latin1`列值：

```sql
Muffler
Müller
MX Systems
MySQL
```

还假设通过以下语句检索列值：

```sql
SELECT X FROM T ORDER BY X COLLATE *collation_name*;
```

使用不同排序规则的`ORDER BY`，以下表格显示了值的排序结果。

| `latin1_swedish_ci` | `latin1_german1_ci` | `latin1_german2_ci` |
| --- | --- | --- |
| 阻尼器 | 阻尼器 | 米勒 |
| MX 系统 | 米勒 | 阻尼器 |
| 米勒 | MX 系统 | MX 系统 |
| MySQL | MySQL | MySQL |

在这个例子中导致不同排序顺序的字符是`ü`（德语“U-umlaut”）。

+   第一列显示了使用瑞典/芬兰排序规则的`SELECT`的结果，该规则表示 U-umlaut 与 Y 排序。

+   第二列显示了使用德国 DIN-1 规则的`SELECT`的结果，该规则表示 U-umlaut 与 U 排序。

+   第三列显示了使用德国 DIN-2 规则的`SELECT`的结果，该规则表示 U-umlaut 与 UE 排序。

**示例 2：搜索德语 Umlauts**

假设您有三个仅由字符集和排序规则不同的表：

```sql
mysql> SET NAMES utf8mb4;
mysql> CREATE TABLE german1 (
         c CHAR(10)
       ) CHARACTER SET latin1 COLLATE latin1_german1_ci;
mysql> CREATE TABLE german2 (
         c CHAR(10)
       ) CHARACTER SET latin1 COLLATE latin1_german2_ci;
mysql> CREATE TABLE germanutf8 (
         c CHAR(10)
       ) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

每个表包含两条记录：

```sql
mysql> INSERT INTO german1 VALUES ('Bar'), ('Bär');
mysql> INSERT INTO german2 VALUES ('Bar'), ('Bär');
mysql> INSERT INTO germanutf8 VALUES ('Bar'), ('Bär');
```

上述排序规则中有两个具有`A = Ä`的相等性，一个没有这种相等性（`latin1_german2_ci`）。因此，比较产生了这里显示的结果：

```sql
mysql> SELECT * FROM german1 WHERE c = 'Bär';
+------+
| c    |
+------+
| Bar  |
| Bär  |
+------+
mysql> SELECT * FROM german2 WHERE c = 'Bär';
+------+
| c    |
+------+
| Bär  |
+------+
mysql> SELECT * FROM germanutf8 WHERE c = 'Bär';
+------+
| c    |
+------+
| Bar  |
| Bär  |
+------+
```

这不是一个错误，而是`latin1_german1_ci`和`utf8mb4_unicode_ci`的排序属性的结果（所示的排序是根据德国 DIN 5007 标准进行的）。
