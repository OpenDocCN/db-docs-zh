> 原文：[`dev.mysql.com/doc/refman/8.0/en/collation-diagnostics.html`](https://dev.mysql.com/doc/refman/8.0/en/collation-diagnostics.html)

#### 12.14.4.3 Index.xml 解析期间的诊断

MySQL 服务器在解析`Index.xml`文件时发现问题时会生成诊断信息：

+   未知标记会被写入错误日志。例如，如果排序规则定义包含`<aaa>`标记，则会产生以下消息：

    ```sql
    [Warning] Buffered warning: Unknown LDML tag:
    'charsets/charset/collation/rules/aaa'
    ```

+   如果排序规则初始化不可能，服务器会报告“未知排序规则”错误，并生成解释问题的警告，就像前面的例子一样。在其他情况下，如果排序规则描述通常正确但包含一些未知标记，排序规则会被初始化并可供使用。未知部分会被忽略，但错误日志中会生成警告。

+   排序规则存在问题会生成警告，客户端可以使用`SHOW WARNINGS`显示这些警告。假设重置规则包含一个长度超过 6 个字符的扩展：

    ```sql
    <reset>abcdefghi</reset>
    <i>x</i>
    ```

    尝试使用排序规则会产生警告：

    ```sql
    mysql> SELECT _utf8mb4'test' COLLATE utf8mb4_test_ci;
    ERROR 1273 (HY000): Unknown collation: 'utf8mb4_test_ci'
    mysql> SHOW WARNINGS;
    +---------+------+----------------------------------------+
    | Level   | Code | Message                                |
    +---------+------+----------------------------------------+
    | Error   | 1273 | Unknown collation: 'utf8mb4_test_ci'   |
    | Warning | 1273 | Expansion is too long at 'abcdefghi=x' |
    +---------+------+----------------------------------------+
    ```
