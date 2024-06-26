# 7.6.6 版本标记

> 原文：[`dev.mysql.com/doc/refman/8.0/en/version-tokens.html`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens.html)

7.6.6.1 版本标记元素

7.6.6.2 安装或卸载版本标记

7.6.6.3 使用版本标记

7.6.6.4 版本标记参考

MySQL 包括版本标记，这是一个功能，可以创建和围绕服务器标记进行同步，应用程序可以使用这些标记来防止访问不正确或过时的数据。

版本标记接口具有以下特征：

+   版本标记是由一个作为键或标识符的名称和一个值组成的对。

+   版本标记可以被锁定。应用程序可以使用标记锁定来指示其他合作应用程序正在使用标记，不应该被修改。

+   版本标记列表是针对每个服务器建立的（例如，用于指定服务器分配或操作状态）。此外，与服务器通信的应用程序可以注册其自己的标记列表，指示其需要服务器处于的状态。应用程序向未处于所需状态的服务器发送的 SQL 语句会产生错误。这是一个信号，告诉应用程序应该寻找一个处于所需状态的不同服务器来接收 SQL 语句。

以下各节描述了版本标记的元素，讨论了如何安装和使用它，并为其元素提供了参考信息。
