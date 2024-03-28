> 原文：[`dev.mysql.com/doc/refman/8.0/en/version-tokens-usage.html`](https://dev.mysql.com/doc/refman/8.0/en/version-tokens-usage.html)

#### 7.6.6.3 使用版本令牌

在使用版本令牌之前，请根据第 7.6.6.2 节，“安装或卸载版本令牌”中提供的说明进行安装。

版本令牌可以派上用场的一个场景是访问一组 MySQL 服务器的系统，但需要通过监视它们并根据负载变化调整服务器分配来进行负载平衡管理。这样的系统包括以下元素：

+   要管理的 MySQL 服务器集合。

+   与服务器通信并将其组织成高可用性组的管理或管理应用程序。组具有不同的目的，每个组内的服务器可能具有不同的分配。某个组内服务器的分配可以随时更改。

+   客户端应用程序访问服务器以检索和更新数据，根据分配给它们的目的选择服务器。例如，客户端不应向只读服务器发送更新。

版本令牌允许根据分配管理服务器访问，而无需客户端反复查询服务器的分配情况：

+   管理应用程序执行服务器分配并在每个服务器上建立版本令牌以反映其分配。该应用程序缓存此信息以提供对其的集中访问点。

    如果管理应用程序在某个时刻需要更改服务器分配（例如，将其从允许写入更改为只读），则更改服务器的版本令牌列表并更新其缓存。

+   为了提高性能，客户端应用程序从管理应用程序获取缓存信息，使它们无需为每个语句检索有关服务器分配的信息。根据其发出的语句类型（例如，读取与写入），客户端选择适当的服务器并连接到它。

+   此外，客户端向服务器发送自己的客户端特定版本令牌，以注册其对服务器所需的分配。对于客户端发送给服务器的每个语句，服务器将其自己的令牌列表与客户端令牌列表进行比较。如果服务器令牌列表包含客户端令牌列表中具有相同值的所有令牌，则存在匹配项，服务器执行该语句。

    另一方面，也许管理应用程序已更改了服务器分配及其版本令牌列表。在这种情况下，新的服务器分配现在可能与客户端要求不兼容。服务器和客户端令牌列表之间存在令牌不匹配，并且服务器在回复语句中返回错误。这是向客户端指示从管理应用程序缓存中刷新其版本令牌信息，并选择一个新服务器进行通信的迹象。

用于检测版本令牌错误并选择新服务器的客户端端逻辑可以以不同的方式实现：

+   客户端可以自行处理所有版本令牌注册、不匹配检测和连接切换。

+   这些操作的逻辑可以在一个连接器中实现，该连接器管理客户端和 MySQL 服务器之间的连接。这样的连接器可能会自行处理不匹配错误检测和语句重发，或者将错误传递给应用程序，由应用程序重新发送语句。

以下示例以更具体的形式说明了前面的讨论。

当版本令牌在给定服务器上初始化时，服务器的版本令牌列表为空。通过调用函数执行令牌列表维护。调用任何版本令牌函数都需要`VERSION_TOKEN_ADMIN`权限（或已弃用的`SUPER`权限），因此预计令牌列表修改将由具有该权限的管理或管理应用程序执行。

假设管理应用程序与一组由客户端查询以访问员工和产品数据库（分别命名为`emp`和`prod`）的服务器通信。所有服务器都被允许处理数据检索语句，但只有一些服务器被允许进行数据库更新。为了在特定于数据库的基础上处理这个问题，管理应用程序在每台服务器上建立了一个版本令牌列表。在给定服务器的令牌列表中，令牌名称代表数据库名称，令牌值为`read`或`write`，取决于数据库必须以只读方式使用还是可以进行读写操作。

客户端应用程序通过设置系统变量注册它们需要服务器匹配的版本令牌列表。变量设置是基于特定客户端的，因此不同的客户端可以注册不同的要求。默认情况下，客户端令牌列表为空，与任何服务器令牌列表匹配。当客户端将其令牌列表设置为非空值时，匹配可能成功或失败，这取决于服务器版本令牌列表。

要为服务器定义版本令牌列表，管理应用程序调用`version_tokens_set()`函数。（还有用于修改和显示令牌列表的函数，稍后描述。）例如，应用程序可能向三个服务器组发送以下语句：

服务器 1:

```sql
mysql> SELECT version_tokens_set('emp=read;prod=read');
+------------------------------------------+
| version_tokens_set('emp=read;prod=read') |
+------------------------------------------+
| 2 version tokens set.                    |
+------------------------------------------+
```

服务器 2:

```sql
mysql> SELECT version_tokens_set('emp=write;prod=read');
+-------------------------------------------+
| version_tokens_set('emp=write;prod=read') |
+-------------------------------------------+
| 2 version tokens set.                     |
+-------------------------------------------+
```

服务器 3:

```sql
mysql> SELECT version_tokens_set('emp=read;prod=write');
+-------------------------------------------+
| version_tokens_set('emp=read;prod=write') |
+-------------------------------------------+
| 2 version tokens set.                     |
+-------------------------------------------+
```

在每种情况下，令牌列表都被指定为以分号分隔的`*`name`*=*`value`*`对的列表。生成的令牌列表值导致以下服务器分配：

+   任何服务器都接受任一数据库的读取。

+   只有服务器 2 接受`emp`数据库的更新。

+   只有服务器 3 接受`prod`数据库的更新���

除了为每个服务器分配版本令牌列表外，管理应用程序还维护反映服务器分配的缓存。

在与服务器通信之前，客户端应用程序会联系管理应用程序并检索有关服务器分配的信息。然后客户端根据这些分配选择服务器。假设客户端想在`emp`数据库上执行读取和写入操作。根据前述分配，只有服务器 2 符合条件。客户端连接到服务器 2 并通过设置其`version_tokens_session`系统变量在那里注册其服务器要求：

```sql
mysql> SET @@SESSION.version_tokens_session = 'emp=write';
```

对于客户端发送到服务器 2 的后续语句，服务器将比较自己的版本令牌列表与客户端列表，以检查它们是否匹配。如果匹配，则语句正常执行：

```sql
mysql> UPDATE emp.employee SET salary = salary * 1.1 WHERE id = 4981;
Query OK, 1 row affected (0.07 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT last_name, first_name FROM emp.employee WHERE id = 4981;
+-----------+------------+
| last_name | first_name |
+-----------+------------+
| Smith     | Abe        |
+-----------+------------+
1 row in set (0.01 sec)
```

服务器和客户端版本令牌列表之间的差异可能以两种方式发生：

+   在`version_tokens_session`值中的令牌名称不在服务器令牌列表中。在这种情况下，会发生`ER_VTOKEN_PLUGIN_TOKEN_NOT_FOUND`错误。

+   在`version_tokens_session`值中的令牌值与服务器令牌列表中相应令牌的值不同。在这种情况下，会发生`ER_VTOKEN_PLUGIN_TOKEN_MISMATCH`错误。

只要服务器 2 的分配不改变，客户端就会继续将其用于读取和写入。但假设管理应用程序想要更改服务器分配，以便将`emp`数据库的写入操作发送到服务器 1 而不是服务器 2。为此，它使用`version_tokens_edit()`来修改两个服务器上的`emp`令牌值（并更新其服务器分配的缓存）：

服务器 1:

```sql
mysql> SELECT version_tokens_edit('emp=write');
+----------------------------------+
| version_tokens_edit('emp=write') |
+----------------------------------+
| 1 version tokens updated.        |
+----------------------------------+
```

服务器 2:

```sql
mysql> SELECT version_tokens_edit('emp=read');
+---------------------------------+
| version_tokens_edit('emp=read') |
+---------------------------------+
| 1 version tokens updated.       |
+---------------------------------+
```

`version_tokens_edit()` 修改服务器令牌列表中命名的令牌，并保持其他令牌不变。

下次客户端向服务器 2 发送语句时，其自身的令牌列表不再与服务器令牌列表匹配，将会发生错误：

```sql
mysql> UPDATE emp.employee SET salary = salary * 1.1 WHERE id = 4982;
ERROR 3136 (42000): Version token mismatch for emp. Correct value read
```

在这种情况下，客户端应联系管理应用程序以获取关于服务器分配的更新信息，选择一个新服务器，并将失败的语句发送到新服务器。

注意

每个客户端必须通过仅发送符合其向特定服务器注册的令牌列表的语句来与版本令牌合作。例如，如果客户端注册了一个令牌列表为`'emp=read'`，那么版本令牌中没有任何内容可以阻止客户端发送对`emp`数据库的更新。客户端必须自行避免这样做。

对于从客户端接收的每个语句，服务器隐式使用锁定，如下所示：

+   对于客户端令牌列表中命名的每个令牌（即在`version_tokens_session`值中），获取共享锁。

+   执行服务器和客户端令牌列表之间的比较

+   根据比较结果执行语句或产生错误

+   释放锁

服务器使用共享锁，以便进行多个会话的比较而不会阻塞，同时防止任何尝试在操作具有相同名称的令牌之前获取独占锁的会话对令牌进行更改。

前面的示例仅使用了版本令牌插件库中包含的一些函数，但还有其他函数。一组函数允许操作和检查服务器的版本令牌列表。另一组函数允许锁定和解锁版本令牌。

这些函数允许创建、更改、删除和检查服务器的版本令牌列表：

+   `version_tokens_set()` 完全替换当前列表并分配新列表。参数是一个以分号分隔的`*`name`*=*`value`*`对列表。

+   `version_tokens_edit()` 允许对当前列表进行部分修改。它可以添加新令牌或更改现有令牌的值。参数是一个以分号分隔的`*`name`*=*`value`*`对列表。

+   `version_tokens_delete()` 从当前列表中删除令牌。参数是一个以分号分隔的令牌名称列表。

+   `version_tokens_show()` 显示当前令牌列表。它不需要参数。

如果成功，这些函数中的每一个都会返回一个指示发生了什么操作的二进制字符串。以下示例建立了服务器令牌列表，通过添加新令牌对其进行修改，删除一些令牌，并显示生成的令牌列表：

```sql
mysql> SELECT version_tokens_set('tok1=a;tok2=b');
+-------------------------------------+
| version_tokens_set('tok1=a;tok2=b') |
+-------------------------------------+
| 2 version tokens set.               |
+-------------------------------------+
mysql> SELECT version_tokens_edit('tok3=c');
+-------------------------------+
| version_tokens_edit('tok3=c') |
+-------------------------------+
| 1 version tokens updated.     |
+-------------------------------+
mysql> SELECT version_tokens_delete('tok2;tok1');
+------------------------------------+
| version_tokens_delete('tok2;tok1') |
+------------------------------------+
| 2 version tokens deleted.          |
+------------------------------------+
mysql> SELECT version_tokens_show();
+-----------------------+
| version_tokens_show() |
+-----------------------+
| tok3=c;               |
+-----------------------+
```

如果令牌列表格式不正确，则会出现警告：

```sql
mysql> SELECT version_tokens_set('tok1=a; =c');
+----------------------------------+
| version_tokens_set('tok1=a; =c') |
+----------------------------------+
| 1 version tokens set.            |
+----------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 42000
Message: Invalid version token pair encountered. The list provided
         is only partially updated. 1 row in set (0.00 sec)
```

如前所述，版本令牌是使用分号分隔的`*`名称`*=*`值`*`对定义的。考虑这个`version_tokens_set()`的调用：

```sql
mysql> SELECT version_tokens_set('tok1=b;;; tok2= a = b ; tok1 = 1\'2 3"4')
+---------------------------------------------------------------+
| version_tokens_set('tok1=b;;; tok2= a = b ; tok1 = 1\'2 3"4') |
+---------------------------------------------------------------+
| 3 version tokens set.                                         |
+---------------------------------------------------------------+
```

版本令牌解释参数如下：

+   名称和值周围的空格将被忽略。允许名称和值内部的空格。（对于`version_tokens_delete()`，它接受没有值的名称列表，名称周围的空格将被忽略。）

+   没有引用机制。

+   令牌的顺序不重要，除非令牌列表包含给定令牌名称的多个实例，最后一个值优先于先前的值。

根据这些规则，前面的`version_tokens_set()`调用会导致一个包含两个令牌的令牌列表：`tok1`的值为`1'2 3"4`，`tok2`的值为`a = b`。要验证这一点，请调用`version_tokens_show()`：

```sql
mysql> SELECT version_tokens_show();
+--------------------------+
| version_tokens_show()    |
+--------------------------+
| tok2=a = b;tok1=1'2 3"4; |
+--------------------------+
```

如果令牌列表包含两个令牌，为什么`version_tokens_set()`返回值为`3 version tokens set`？这是因为原始令牌列表包含了两个`tok1`的定义，第二个定义替换了第一个。

版本令牌的令牌操作函数对令牌名称和值施加了以下约束：

+   令牌名称不能包含`=`��`；`字符，最大长度为 64 个字符。

+   令牌值不能包含`；`字符。值的长度受`max_allowed_packet`系统变量值的限制。

+   版本令牌将令牌名称和值视为二进制字符串，因此比较区分大小写。

版本令牌还包括一组函数，使令牌可以被锁定和解锁：

+   `version_tokens_lock_exclusive()`获取独占版本令牌锁。它接受一个或多个锁名称列表和一个超时值。

+   `version_tokens_lock_shared()`获取共享版本令牌锁。它接受一个或多个锁名称列表和一个超时值。

+   `version_tokens_unlock()`释放版本令牌锁（独占和共享）。它不接受参数。

每个锁定函数返回非零值表示成功。否则，将发生错误：

```sql
mysql> SELECT version_tokens_lock_shared('lock1', 'lock2', 0);
+-------------------------------------------------+
| version_tokens_lock_shared('lock1', 'lock2', 0) |
+-------------------------------------------------+
|                                               1 |
+-------------------------------------------------+

mysql> SELECT version_tokens_lock_shared(NULL, 0);
ERROR 3131 (42000): Incorrect locking service lock name '(null)'.
```

使用版本令牌锁定函数进行锁定是建议性的；应用程序必须同意合作。

可以锁定不存在的令牌名称。这不会创建令牌。

注意

版本令牌锁定函数基于第 7.6.9.1 节，“锁定服务”中描述的锁定服务，并且对于共享和独占锁具有相同的语义。（版本令牌使用内置于服务器中的锁定服务例程，而不是锁定服务函数接口，因此不需要安装这些函数来使用版本令牌。）版本令牌获取的锁使用 `version_token_locks` 的锁定服务命名空间。可以使用性能模式监视锁定服务锁，因此对于版本令牌锁也是如此。有关详细信息，请参见锁定服务监控。

对于版本令牌锁定函数，令牌名称参数将按照指定的方式使用。周围的空白不会被忽略，`=` 和 `;` 字符是允许的。这是因为版本令牌只是将要锁定的令牌名称原样传递给锁定服务。