# 14.24 精度数学

> 原文：[`dev.mysql.com/doc/refman/8.0/en/precision-math.html`](https://dev.mysql.com/doc/refman/8.0/en/precision-math.html)

14.24.1 数值类型

14.24.2 DECIMAL 数据类型特性

14.24.3 表达式处理

14.24.4 四舍五入行为

14.24.5 精度数学示例

MySQL 提供了对精度数学的支持：处理数字值以获得极其准确的结果，并对无效值具有高度控制。精度数学基于这两个特性：

+   控制服务器对接受或拒绝无效数据的严格程度的 SQL 模式。

+   MySQL 固定点算术库。

这些特性对数字操作有几个影响，并提供了与标准 SQL 高度一致性：

+   **精确计算**：对于精确值数字，计算不会引入浮点错误。相反，使用精确精度。例如，MySQL 将`.0001`这样的数字视为精确值，而不是近似值，将其累加 10,000 次的结果确切为`1`，而不是仅仅“接近”1 的值。

+   **明确定义的四舍五入行为**：对于精确值数字，`ROUND()`的结果取决于其参数，而不取决于环境因素，如底层 C 库的工作方式。

+   **平台独立性**：对于精确数值的操作在不同平台（如 Windows 和 Unix）上是相同的。

+   **对无效值处理的控制**：溢出和除零可检测并可视为错误处理。例如，您可以将列值过大视为错误，而不是将值截断为列数据类型范围内。同样，您可以将除零视为错误，而不是产生`NULL`结果的操作。采取哪种方法由服务器 SQL 模式设置决定。

以下讨论涵盖了精度数学的几个方面，包括与旧应用程序可能存在的不兼容性。最后，给出了一些示例，展示了 MySQL 如何精确处理数字操作。有关控制 SQL 模式的信息，请参见第 7.1.11 节，“服务器 SQL 模式”。
