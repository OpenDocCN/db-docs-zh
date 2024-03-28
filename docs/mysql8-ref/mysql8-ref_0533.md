> 原文：[`dev.mysql.com/doc/refman/8.0/en/function-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/function-optimization.html)

#### 10.2.1.20 函数调用优化

MySQL 函数在内部被标记为确定性或非确定性。如果给定其参数的固定值，函数在不同调用中可以返回不同结果，则该函数是非确定性的。非确定性函数的示例：`RAND()`，`UUID()`。

如果一个函数被标记为非确定性，在`WHERE`子句中对其进行引用会对每一行（从一个表中选择）或每一行组合（从多表连接中选择）进行评估。

MySQL 还根据参数类型确定何时评估函数，参数是表列还是常量值。将表列作为参数的确定性函数必须在该列更改值时进行评估。

非确定性函数可能会影响查询性能。例如，某些优化可能不可用，或者可能需要更多的锁定。以下讨论使用`RAND()`，但也适用于其他非确定性函数。

假设表`t`有以下定义：

```sql
CREATE TABLE t (id INT NOT NULL PRIMARY KEY, col_a VARCHAR(100));
```

考虑以下两个查询：

```sql
SELECT * FROM t WHERE id = POW(1,2);
SELECT * FROM t WHERE id = FLOOR(1 + RAND() * 49);
```

由于对主键的等式比较，这两个查询似乎都使用主键查找，但这只对第一个查询是真的：

+   第一个查询始终最多生成一行，因为具有常量参数的`POW()`是一个常量值，并用于索引查找。

+   第二个查询包含一个使用非确定性函数`RAND()`的表达式，该函数在查询中不是常量，而是实际上对表`t`的每一行都有一个新值。因此，查询读取表的每一行，为每一行评估谓词，并输出所有主键与随机值匹配的行。这可能是零行、一行或多行，取决于`id`列值和`RAND()`序列中的值。

非确定性的影响不仅限于`SELECT`语句。此`UPDATE`语句使用非确定性函数选择要修改的行：

```sql
UPDATE t SET col_a = *some_expr* WHERE id = FLOOR(1 + RAND() * 49);
```

大概意图是更新与表达式匹配的主键最多一行。但是，根据`id`列值和`RAND()`序列中的值，可能会更新零行、一行或多行。

刚才描述的行为对性能和复制有影响：

+   因为非确定性函数不会产生常量值，优化器无法使用可能适用的策略，如索引查找。结果可能是表扫描。

+   `InnoDB`可能会升级为范围键锁，而不是为一个匹配行获取单行锁。

+   不确定性地执行的更新对复制是不安全的。

困难源于`RAND()`函数对表中每一行都进行一次评估。为避免多次函数评估，可以使用以下技术之一：

+   将包含非确定性函数的表达式移动到单独的语句中，并将值保存在变量中。在原始语句中，用对变量的引用替换表达式，优化器可以将其视为常量值：

    ```sql
    SET @keyval = FLOOR(1 + RAND() * 49);
    UPDATE t SET col_a = *some_expr* WHERE id = @keyval;
    ```

+   将随机值分配给派生表中的变量。这种技术会导致变量在比较`WHERE`子句中使用之前被分配一个值，仅一次：

    ```sql
    UPDATE /*+ NO_MERGE(dt) */ t, (SELECT FLOOR(1 + RAND() * 49) AS r) AS dt
    SET col_a = *some_expr* WHERE id = dt.r;
    ```

如前所述，在`WHERE`子句中的非确定性表达式可能会阻止优化并导致表扫描。然而，如果其他表达式是确定性的，可能可以部分优化`WHERE`子句。例如：

```sql
SELECT * FROM t WHERE partial_key=5 AND some_column=RAND();
```

如果优化器可以使用`partial_key`来减少选择的行集，那么`RAND()`的执行次数会减少，从而减少非确定性对优化的影响。
