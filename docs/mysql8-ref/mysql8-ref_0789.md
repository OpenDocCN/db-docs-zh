# 14.1 内置函数和运算符参考

> 原文：[`dev.mysql.com/doc/refman/8.0/en/built-in-function-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/built-in-function-reference.html)

以下表格列出了每个内置（原生）函数和运算符，并提供了每个的简短描述。有关在运行时可加载的函数列表，请参见第 14.2 节，“可加载函数参考”。

**表格 14.1 内置函数和运算符**

| 名称 | 描述 | 引入 | 废弃 |
| --- | --- | --- | --- |
| `&` | 按位与 |  |  |
| `>` | 大于运算符 |  |  |
| `>>` | 右移 |  |  |
| `>=` | 大于或等于运算符 |  |  |
| `<` | 小于运算符 |  |  |
| `<>`, `!=` | 不等于运算符 |  |  |
| `<<` | 左移 |  |  |
| `<=` | 小于或等于运算符 |  |  |
| `<=>` | NULL 安全等于运算符 |  |  |
| `%`, `MOD` | 取模运算符 |  |  |
| `*` | 乘法运算符 |  |  |
| `+` | 加法运算符 |  |  |
| `-` | 减法运算符 |  |  |
| `-` | 改变参数的符号 |  |  |
| `->` | 在评估路径后从 JSON 列返回值；相当于 JSON_EXTRACT()。 |  |  |
| `->>` | 在评估路径并取消引用结果后从 JSON 列返回值；相当于 JSON_UNQUOTE(JSON_EXTRACT())。 |  |  |
| `/` | 除法运算符 |  |  |
| `:=` | 赋值 |  |  |
| `=` | 赋值（作为 `SET` 语句的一部分，或作为 `UPDATE` 语句中 `SET` 子句的一部分） |  |  |
| `=` | 等于运算符 |  |  |
| `^` | 按位异或 |  |  |
| `ABS()` | 返回绝对值 |  |  |
| `ACOS()` | 返回反余弦 |  |  |
| `ADDDATE()` | 将时间值（间隔）添加到日期值中 |  |  |
| `ADDTIME()` | 添加时间 |  |  |
| `AES_DECRYPT()` | 使用 AES 解密 |  |  |
| `AES_ENCRYPT()` | 使用 AES 加密 |  |  |
| `AND`, `&&` | 逻辑与 |  |  |
| `ANY_VALUE()` | 抑制 ONLY_FULL_GROUP_BY 值拒绝 |  |  |
| `ASCII()` | 返回最左字符的数字值 |  |  |
| `ASIN()` | 返回反正弦 |  |  |
| `asynchronous_connection_failover_add_managed()` | 将组成员源服务器配置信息添加到复制通道源列表 | 8.0.23 |  |
| `asynchronous_connection_failover_add_source()` | 将源服务器配置信息添加到复制通道源列表 | 8.0.22 |  |
| `asynchronous_connection_failover_delete_managed()` | 从复制通道源列表中删除受管组 | 8.0.23 |  |
| `asynchronous_connection_failover_delete_source()` | 从复制通道源列表中删除源服务器 | 8.0.22 |  |
| `asynchronous_connection_failover_reset()` | 删除与组复制异步故障转移相关的所有设置 | 8.0.27 |  |
| `ATAN()` | 返回反正切 |  |  |
| `ATAN2()`, `ATAN()` | 返回两个参数的反正切 |  |  |
| `AVG()` | 返回参数的平均值 |  |  |
| `BENCHMARK()` | 反复执行表达式 |  |  |
| `BETWEEN ... AND ...` | 值是否在一系列值范围内 |  |  |
| `BIN()` | 返回包含数字的二进制表示的字符串 |  |  |
| `BIN_TO_UUID()` | 将二进制 UUID 转换为字符串 |  |  |
| `BINARY` | 将字符串转换为二进制字符串 |  | 8.0.27 |
| `BIT_AND()` | 返回按位与 |  |  |
| `BIT_COUNT()` | 返回设置的位数 |  |  |
| `BIT_LENGTH()` | 返回参数的位长度 |  |  |
| `BIT_OR()` | 返回按位或 |  |  |
| `BIT_XOR()` | 返回按位异或 |  |  |
| `CAN_ACCESS_COLUMN()` | 仅供内部使用 |  |  |
| `CAN_ACCESS_DATABASE()` | 仅供内部使用 |  |  |
| `CAN_ACCESS_TABLE()` | 仅供内部使用 |  |  |
| `CAN_ACCESS_USER()` | 仅供内部使用 | 8.0.22 |  |
| `CAN_ACCESS_VIEW()` | 仅供内部使用 |  |  |
| `CASE` | Case 运算符 |  |  |
| `CAST()` | 将值转换为特定类型 |  |  |
| `CEIL()` | 返回不小于参数的最小整数值 |  |  |
| `CEILING()` | 返回不小于参数的最小整数值 |  |  |
| `CHAR()` | 返回每个传递的整数的字符 |  |  |
| `CHAR_LENGTH()` | 返回参数中的字符数 |  |  |
| `CHARACTER_LENGTH()` | CHAR_LENGTH()的同义词 |  |  |
| `CHARSET()` | 返回参数的字符集 |  |  |
| `COALESCE()` | 返回第一个非 NULL 参数 |  |  |
| `COERCIBILITY()` | 返回字符串参数的排序强制性值 |  |  |
| `COLLATION()` | 返回字符串参数的排序规则 |  |  |
| `COMPRESS()` | 返回结果作为二进制字符串 |  |  |
| `CONCAT()` | 返回连接的字符串 |  |  |
| `CONCAT_WS()` | 返回带有分隔符的连接 |  |  |
| `CONNECTION_ID()` | 返回连接的连接 ID（线程 ID） |  |  |
| `CONV()` | 在不同进制之间转换数字 |  |  |
| `CONVERT()` | 将值转换为特定类型 |  |  |
| `CONVERT_TZ()` | 从一个时区转换到另一个时区 |  |  |
| `COS()` | 返回余弦值 |  |  |
| `COT()` | 返回余切 |  |  |
| `COUNT()` | 返回返回的行数计数 |  |  |
| `COUNT(DISTINCT)` | 返回不同值的计数 |  |  |
| `CRC32()` | 计算循环冗余校验值 |  |  |
| `CUME_DIST()` | 累积分布值 |  |  |
| `CURDATE()` | 返回当前日期 |  |  |
| `CURRENT_DATE()`, `CURRENT_DATE` | CURDATE()的同义词 |  |  |
| `CURRENT_ROLE()` | 返回当前活动角色 |  |  |
| `CURRENT_TIME()`, `CURRENT_TIME` | CURTIME()的同义词 |  |  |
| `CURRENT_TIMESTAMP()`, `CURRENT_TIMESTAMP` | NOW()的同义词 |  |  |
| `CURRENT_USER()`, `CURRENT_USER` | 认证用户名称和主机名 |  |  |
| `CURTIME()` | 返回当前时间 |  |  |
| `DATABASE()` | 返回默认（当前）数据库名称 |  |  |
| `DATE()` | 提取日期或日期时间表达式的日期部分 |  |  |
| `DATE_ADD()` | 将时间值（间隔）添加到日期值 |  |  |
| `DATE_FORMAT()` | 格式化指定的日期 |  |  |
| `DATE_SUB()` | 从日期中减去时间值（间隔） |  |  |
| `DATEDIFF()` | 减去两个日期 |  |  |
| `DAY()` | DAYOFMONTH()的同义词 |  |  |
| `DAYNAME()` | 返回星期几的名称 |  |  |
| `DAYOFMONTH()` | 返回月份的日期（0-31） |  |  |
| `DAYOFWEEK()` | 返回参数的星期索引 |  |  |
| `DAYOFYEAR()` | 返回年份的日期（1-366） |  |  |
| `DEFAULT()` | 返回表列的默认值 |  |  |
| `DEGREES()` | 将弧度转换为度数 |  |  |
| `DENSE_RANK()` | 在其分区内当前行的排名，无间隔 |  |  |
| `DIV` | 整数除法 |  |  |
| `ELT()` | 返回索引号处的字符串 |  |  |
| `EXP()` | 求幂 |  |  |
| `EXPORT_SET()` | 返回一个字符串，对于值位中的每个位设置，您会得到一个打开字符串，对于每个未设置的位，您会得到一个关闭字符串 |  |  |
| `EXTRACT()` | 提取日期的部分 |  |  |
| `ExtractValue()` | 使用 XPath 表示法从 XML 字符串中提取值 |  |  |
| `FIELD()` | 第一个参数在后续参数中的索引（位置） |  |  |
| `FIND_IN_SET()` | 第一个参数在第二个参数中的索引（位置） |  |  |
| `FIRST_VALUE()` | 窗口帧的第一行中的参数值 |  |  |
| `FLOOR()` | 返回不大于参数的最大整数值 |  |  |
| `FORMAT()` | 返回格式化为指定小数位数的数字 |  |  |
| `FORMAT_BYTES()` | 将字节计数转换为带单位的值 | 8.0.16 |  |
| `FORMAT_PICO_TIME()` | 将皮秒时间转换为带单位的值 | 8.0.16 |  |
| `FOUND_ROWS()` | 对于带有 LIMIT 子句的 SELECT 语句，如果没有 LIMIT 子句，将返回的行数 |  |  |
| `FROM_BASE64()` | 解码 base64 编码的字符串并返回结果 |  |  |
| `FROM_DAYS()` | 将天数转换为日期 |  |  |
| `FROM_UNIXTIME()` | 将 Unix 时间戳格式化为日期 |  |  |
| `GeomCollection()` | 从几何体构造几何集合 |  |  |
| `GeometryCollection()` | 从几何体构造几何集合 |  |  |
| `GET_DD_COLUMN_PRIVILEGES()` | 仅供内部使用 |  |  |
| `GET_DD_CREATE_OPTIONS()` | 仅供内部使用 |  |  |
| `GET_DD_INDEX_SUB_PART_LENGTH()` | 仅供内部使用 |  |  |
| `GET_FORMAT()` | 返回日期格式字符串 |  |  |
| `GET_LOCK()` | 获取命名锁 |  |  |
| `GREATEST()` | 返回最大的参数 |  |  |
| `GROUP_CONCAT()` | 返回连接的字符串 |  |  |
| `group_replication_disable_member_action()` | 为指定事件禁用成员操作 | 8.0.26 |  |
| `group_replication_enable_member_action()` | 为指定事件启用成员操作 | 8.0.26 |  |
| `group_replication_get_communication_protocol()` | 获取当前正在使用的组复制通信协议的版本 | 8.0.16 |  |
| `group_replication_get_write_concurrency()` | 获取当前为组设置的最大一致性实例数 | 8.0.13 |  |
| `group_replication_reset_member_actions()` | 将所有成员操作重置为默认值，并将配置版本号重置为 1 | 8.0.26 |  |
| `group_replication_set_as_primary()` | 将特定组成员设为主节点 | 8.0.29 |  |
| `group_replication_set_communication_protocol()` | 设置组复制通信协议要使用的版本 | 8.0.16 |  |
| `group_replication_set_write_concurrency()` | 设置可以并行执行的最大一致性实例数 | 8.0.13 |  |
| `group_replication_switch_to_multi_primary_mode()` | 将运行在单主模式下的组的模式更改为多主模式 | 8.0.13 |  |
| `group_replication_switch_to_single_primary_mode()` | 将运行在多主模式下的组的模式更改为单主模式 | 8.0.13 |  |
| `GROUPING()` | 区分超级聚合 ROLLUP 行和常规行 |  |  |
| `GTID_SUBSET()` | 如果子集中的所有 GTID 也在集合中，则返回 true；否则返回 false。 |  |  |
| `GTID_SUBTRACT()` | 返回集合中不在子集中的所有 GTID。 |  |  |
| `HEX()` | 十六进制表示的十进制或字符串值 |  |  |
| `HOUR()` | 提取小时 |  |  |
| `ICU_VERSION()` | ICU 库版本 |  |  |
| `IF()` | 如果/否则结构 |  |  |
| `IFNULL()` | 如果/否则结构中的空值 |  |  |
| `IN()` | 值是否在一组值内 |  |  |
| `INET_ATON()` | 返回 IP 地址的数值 |  |  |
| `INET_NTOA()` | 从数值返回 IP 地址 |  |  |
| `INET6_ATON()` | 返回 IPv6 地址的数值 |  |  |
| `INET6_NTOA()` | 从数值返回 IPv6 地址 |  |  |
| `INSERT()` | 在指定位置插入子字符串，最多指定字符数 |  |  |
| `INSTR()` | 返回子字符串第一次出现的索引 |  |  |
| `INTERNAL_AUTO_INCREMENT()` | 仅供内部使用 |  |  |
| `INTERNAL_AVG_ROW_LENGTH()` | 仅供内部使用 |  |  |
| `INTERNAL_CHECK_TIME()` | 仅供内部使用 |  |  |
| `INTERNAL_CHECKSUM()` | 仅供内部使用 |  |  |
| `INTERNAL_DATA_FREE()` | 仅供内部使用 |  |  |
| `INTERNAL_DATA_LENGTH()` | 仅供内部使用 |  |  |
| `INTERNAL_DD_CHAR_LENGTH()` | 仅供内部使用 |  |  |
| `INTERNAL_GET_COMMENT_OR_ERROR()` | 仅供内部使用 |  |  |
| `INTERNAL_GET_ENABLED_ROLE_JSON()` | 仅供内部使用 | 8.0.19 |  |
| `INTERNAL_GET_HOSTNAME()` | 仅供内部使用 | 8.0.19 |  |
| `INTERNAL_GET_USERNAME()` | 仅供内部使用 | 8.0.19 |  |
| `INTERNAL_GET_VIEW_WARNING_OR_ERROR()` | 仅供内部使用 |  |  |
| `INTERNAL_INDEX_COLUMN_CARDINALITY()` | 仅供内部使用 |  |  |
| `INTERNAL_INDEX_LENGTH()` | 仅供内部使用 |  |  |
| `INTERNAL_IS_ENABLED_ROLE()` | 仅供内部使用 | 8.0.19 |  |
| `INTERNAL_IS_MANDATORY_ROLE()` | 仅供内部使用 | 8.0.19 |  |
| `INTERNAL_KEYS_DISABLED()` | 仅供内部使用 |  |  |
| `INTERNAL_MAX_DATA_LENGTH()` | 仅供内部使用 |  |  |
| `INTERNAL_TABLE_ROWS()` | 仅供内部使用 |  |  |
| `INTERNAL_UPDATE_TIME()` | 仅供内部使用 |  |  |
| `INTERVAL()` | 返回小于第一个参数的参数的索引 |  |  |
| `IS` | 测试一个值是否为布尔值 |  |  |
| `IS_FREE_LOCK()` | 检查指定的锁是否空闲 |  |  |
| `IS_IPV4()` | 参数是否为 IPv4 地址 |  |  |
| `IS_IPV4_COMPAT()` | 参数是否为 IPv4 兼容地址 |  |  |
| `IS_IPV4_MAPPED()` | 参数是否为 IPv4 映射地址 |  |  |
| `IS_IPV6()` | 参数是否为 IPv6 地址 |  |  |
| `IS NOT` | 测试一个值是否为布尔值 |  |  |
| `IS NOT NULL` | 非 NULL 值测试 |  |  |
| `IS NULL` | NULL 值测试 |  |  |
| `IS_USED_LOCK()` | 检查指定的锁是否正在使用；如果是，则返回连接标识符 |  |  |
| `IS_UUID()` | 参数是否为有效的 UUID |  |  |
| `ISNULL()` | 测试参数是否为 NULL |  |  |
| `JSON_ARRAY()` | 创建 JSON 数组 |  |  |
| `JSON_ARRAY_APPEND()` | 向 JSON 文档追加数据 |  |  |
| `JSON_ARRAY_INSERT()` | 插入到 JSON 数组中 |  |  |
| `JSON_ARRAYAGG()` | 将结果集作为单个 JSON 数组返回 |  |  |
| `JSON_CONTAINS()` | JSON 文档是否包含特定路径处的对象 |  |  |
| `JSON_CONTAINS_PATH()` | JSON 文档是否在路径处包含任何数据 |  |  |
| `JSON_DEPTH()` | JSON 文档的最大深度 |  |  |
| `JSON_EXTRACT()` | 从 JSON 文档中返回数据 |  |  |
| `JSON_INSERT()` | 将数据插入 JSON 文档 |  |  |
| `JSON_KEYS()` | JSON 文档中的键数组 |  |  |
| `JSON_LENGTH()` | JSON 文档中的元素数量 |  |  |
| `JSON_MERGE()` | 合并 JSON 文档，保留重复的键。已弃用的 JSON_MERGE_PRESERVE()的同义词 |  | 是 |
| `JSON_MERGE_PATCH()` | 合并 JSON 文档，替换重复键的值 |  |  |
| `JSON_MERGE_PRESERVE()` | 合并 JSON 文档，保留重复的键 |  |  |
| `JSON_OBJECT()` | 创建 JSON 对象 |  |  |
| `JSON_OBJECTAGG()` | 将结果集作为单个 JSON 对象返回 |  |  |
| `JSON_OVERLAPS()` | 比较两个 JSON 文档，如果它们有任何公共键值对或数组元素，则返回 TRUE（1），否则返回 FALSE（0） | 8.0.17 |  |
| `JSON_PRETTY()` | 以人类可读的格式打印 JSON 文档 |  |  |
| `JSON_QUOTE()` | 引用 JSON 文档 |  |  |
| `JSON_REMOVE()` | 从 JSON 文档中删除数据 |  |  |
| `JSON_REPLACE()` | 替换 JSON 文档中的值 |  |  |
| `JSON_SCHEMA_VALID()` | 针对 JSON 模式验证 JSON 文档；如果文档符合模式，则返回 TRUE/1，否则返回 FALSE/0 | 8.0.17 |  |
| `JSON_SCHEMA_VALIDATION_REPORT()` | 针对 JSON 模式验证 JSON 文档；返回 JSON 格式的验证结果报告，包括成功或失败以及失败原因 | 8.0.17 |  |
| `JSON_SEARCH()` | JSON 文档中值的路径 |  |  |
| `JSON_SET()` | 将数据插入 JSON 文档 |  |  |
| `JSON_STORAGE_FREE()` | 部分更新后 JSON 列值的二进制表示中释放的空间 |  |  |
| `JSON_STORAGE_SIZE()` | 用于存储 JSON 文档的二进制表示的空间 |  |  |
| `JSON_TABLE()` | 将 JSON 表达式的数据作为关系表返回 |  |  |
| `JSON_TYPE()` | JSON 值的类型 |  |  |
| `JSON_UNQUOTE()` | 取消 JSON 值的引号 |  |  |
| `JSON_VALID()` | JSON 值是否有效 |  |  |
| `JSON_VALUE()` | 从 JSON 文档中提取路径指向的值；将该值作为 VARCHAR(512) 或指定类型返回 | 8.0.21 |  |
| `LAG()` | 分区内当前行后退行的参数值 |  |  |
| `LAST_DAY` | 返回参数所在月份的最后一天 |  |  |
| `LAST_INSERT_ID()` | 最后一次 INSERT 的 AUTOINCREMENT 列的值 |  |  |
| `LAST_VALUE()` | 窗口帧中最后一行的参数值 |  |  |
| `LCASE()` | LOWER() 的同义词 |  |  |
| `LEAD()` | 分区内当前行前导行的参数值 |  |  |
| `LEAST()` | 返回最小的参数 |  |  |
| `LEFT()` | 返回指定数量的最左侧字符 |  |  |
| `LENGTH()` | 返回字符串的字节长度 |  |  |
| `LIKE` | 简单的模式匹配 |  |  |
| `LineString()` | 从 Point 值构造 LineString |  |  |
| `LN()` | 返回参数的自然对数 |  |  |
| `LOAD_FILE()` | 加载指定文件 |  |  |
| `LOCALTIME()`, `LOCALTIME` | NOW() 的同义词 |  |  |
| `LOCALTIMESTAMP`, `LOCALTIMESTAMP()` | NOW() 的同义词 |  |  |
| `LOCATE()` | 返回子字符串第一次出现的位置 |  |  |
| `LOG()` | 返回第一个参数的自然对数 |  |  |
| `LOG10()` | 返回参数的以 10 为底的对数 |  |  |
| `LOG2()` | 返回参数的以 2 为底的对数 |  |  |
| `LOWER()` | 返回小写参数 |  |  |
| `LPAD()` | 返回左侧填充了指定字符串的字符串参数 |  |  |
| `LTRIM()` | 移除前导空格 |  |  |
| `MAKE_SET()` | 返回一组逗号分隔的字符串，其中对应位在位中设置 |  |  |
| `MAKEDATE()` | 从年份和一年中的天数创建日期 |  |  |
| `MAKETIME()` | 从小时、分钟、秒创建时间 |  |  |
| `MASTER_POS_WAIT()` | 阻塞，直到副本读取并应用到指定位置的所有更新 |  | 8.0.26 |
| `MATCH()` | 执行全文搜索 |  |  |
| `MAX()` | 返回最大值 |  |  |
| `MBRContains()` | 一个几何图形的 MBR 是否包含另一个几何图形的 MBR |  |  |
| `MBRCoveredBy()` | 一个 MBR 是否被另一个覆盖 |  |  |
| `MBRCovers()` | 一个 MBR 是否覆盖另一个 |  |  |
| `MBRDisjoint()` | 两个几何图形的 MBR 是否不相交 |  |  |
| `MBREquals()` | 两个几何图形的 MBR 是否相等 |  |  |
| `MBRIntersects()` | 两个几何图形的 MBR 是否相交 |  |  |
| `MBROverlaps()` | 两个几何图形的 MBR 是否重叠 |  |  |
| `MBRTouches()` | 两个几何图形的 MBR 是否相接触 |  |  |
| `MBRWithin()` | 一个几何图形的 MBR 是否在另一个几何图形的 MBR 内部 |  |  |
| `MD5()` | 计算 MD5 校验和 |  |  |
| `MEMBER OF()` | 如果第一个操作数与作为第二个操作数传递的 JSON 数组中的任何元素匹配，则返回 true（1），否则返回 false（0） | 8.0.17 |  |
| `MICROSECOND()` | 从参数返回微秒 |  |  |
| `MID()` | 从指定位置开始返回子字符串 |  |  |
| `MIN()` | 返回最小值 |  |  |
| `MINUTE()` | 从参数返回分钟 |  |  |
| `MOD()` | 返回余数 |  |  |
| `MONTH()` | 返回传递日期的月份 |  |  |
| `MONTHNAME()` | 返回月份的名称 |  |  |
| `MultiLineString()` | 从 LineString 值构造多线 |  |  |
| `MultiPoint()` | 从点值构造多点 |  |  |
| `MultiPolygon()` | 从多边形值构造多边形 |  |  |
| `NAME_CONST()` | 使列具有给定名称 |  |  |
| `NOT`, `!` | 反转值 |  |  |
| `NOT BETWEEN ... AND ...` | 值是否不在一组值的范围内 |  |  |
| `NOT IN()` | 值是否不在一组值内 |  |  |
| `NOT LIKE` | 简单模式匹配的否定 |  |  |
| `NOT REGEXP` | REGEXP 的否定 |  |  |
| `NOW()` | 返回当前日期和时间 |  |  |
| `NTH_VALUE()` | 窗口帧的第 N 行参数值 |  |  |
| `NTILE()` | 当前行在其分区内的桶号 |  |  |
| `NULLIF()` | 如果 expr1 = expr2 则返回 NULL |  |  |
| `OCT()` | 返回包含数字的八进制表示的字符串 |  |  |
| `OCTET_LENGTH()` | LENGTH() 的同义词 |  |  |
| `OR`, `&#124;&#124;` | 逻辑或 |  |  |
| `ORD()` | 返回参数的最左字符的字符代码 |  |  |
| `PERCENT_RANK()` | 百分比排名值 |  |  |
| `PERIOD_ADD()` | 将一个周期添加到年-月 |  |  |
| `PERIOD_DIFF()` | 返回两个周期之间的月数 |  |  |
| `PI()` | 返回圆周率的值 |  |  |
| `Point()` | 从坐标构造点 |  |  |
| `Polygon()` | 从 LineString 参数构造多边形 |  |  |
| `POSITION()` | LOCATE() 的同义词 |  |  |
| `POW()` | 返回参数的指定幂次方 |  |  |
| `POWER()` | 返回参数的指定幂次方 |  |  |
| `PS_CURRENT_THREAD_ID()` | 当前线程的性能模式线程 ID | 8.0.16 |  |
| `PS_THREAD_ID()` | 给定线程的性能模式线程 ID | 8.0.16 |  |
| `QUARTER()` | 返回日期参数的季度 |  |  |
| `QUOTE()` | 为在 SQL 语句中使用而转义参数 |  |  |
| `RADIANS()` | 将参数转换为弧度 |  |  |
| `RAND()` | 返回一个随机浮点值 |  |  |
| `RANDOM_BYTES()` | 返回一个随机字节向量 |  |  |
| `RANK()` | 当前行在其分区内的排名，带有间隔 |  |  |
| `REGEXP` | 字符串是否匹配正则表达式 |  |  |
| `REGEXP_INSTR()` | 匹配正则表达式的子字符串的起始索引 |  |  |
| `REGEXP_LIKE()` | 字符串是否匹配正则表达式 |  |  |
| `REGEXP_REPLACE()` | 替换匹配正则表达式的子字符串 |  |  |
| `REGEXP_SUBSTR()` | 返回匹配正则表达式的子字符串 |  |  |
| `RELEASE_ALL_LOCKS()` | 释放所有当前命名的锁 |  |  |
| `RELEASE_LOCK()` | 释放命名的锁 |  |  |
| `REPEAT()` | 将字符串重复指定次数 |  |  |
| `REPLACE()` | 替换指定字符串的出现次数 |  |  |
| `REVERSE()` | 反转字符串中的字符 |  |  |
| `RIGHT()` | 返回指定右侧字符数 |  |  |
| `RLIKE` | 字符串是否匹配正则表达式 |  |  |
| `ROLES_GRAPHML()` | 返回表示内存角色子图的 GraphML 文档 |  |  |
| `ROUND()` | 四舍五入参数 |  |  |
| `ROW_COUNT()` | 更新的行数 |  |  |
| `ROW_NUMBER()` | 当前行在其分区内的编号 |  |  |
| `RPAD()` | 将字符串重复指定次数 |  |  |
| `RTRIM()` | 移除字符串末尾的空格 |  |  |
| `SCHEMA()` | DATABASE()的同义词 |  |  |
| `SEC_TO_TIME()` | 将秒数转换为'hh:mm:ss'格式 |  |  |
| `SECOND()` | 返回秒数 (0-59) |  |  |
| `SESSION_USER()` | USER()的同义词 |  |  |
| `SHA1()`, `SHA()` | 计算 SHA-1 160 位校验和 |  |  |
| `SHA2()` | 计算 SHA-2 校验和 |  |  |
| `SIGN()` | 返回参数的符号 |  |  |
| `SIN()` | 返回参数的正弦值 |  |  |
| `SLEEP()` | 休眠指定秒数 |  |  |
| `SOUNDEX()` | 返回一个 soundex 字符串 |  |  |
| `SOUNDS LIKE` | 比较音频 |  |  |
| `SOURCE_POS_WAIT()` | 阻塞直到复制品读取并应用到指定位置的所有更新 | 8.0.26 |  |
| `SPACE()` | 返回指定数量的空格字符串 |  |  |
| `SQRT()` | 返回参数的平方根 |  |  |
| `ST_Area()` | 返回多边形或多重多边形的面积 |  |  |
| `ST_AsBinary()`, `ST_AsWKB()` | 从内部几何格式转换为 WKB |  |  |
| `ST_AsGeoJSON()` | 从几何体生成 GeoJSON 对象 |  |  |
| `ST_AsText()`, `ST_AsWKT()` | 从内部几何格式转换为 WKT |  |  |
| `ST_Buffer()` | 返回距离给定几何体一定距离内的点的几何体 |  |  |
| `ST_Buffer_Strategy()` | 为 ST_Buffer()生成策略选项 |  |  |
| `ST_Centroid()` | 返回几何体的质心点 |  |  |
| `ST_Collect()` | 将空间值聚合为集合 | 8.0.24 |  |
| `ST_Contains()` | 判断一个几何体是否包含另一个 |  |  |
| `ST_ConvexHull()` | 返回几何体的凸包 |  |  |
| `ST_Crosses()` | 一个几何体是否穿过另一个 |  |  |
| `ST_Difference()` | 两个几何体的点集差 |  |  |
| `ST_Dimension()` | 几何体的维度 |  |  |
| `ST_Disjoint()` | 一个几何体是否与另一个不相交 |  |  |
| `ST_Distance()` | 一个几何体到另一个几何体的距离 |  |  |
| `ST_Distance_Sphere()` | 两个几何体在地球上的最小距离 |  |  |
| `ST_EndPoint()` | 线串的终点 |  |  |
| `ST_Envelope()` | 返回几何体的最小边界矩形 |  |  |
| `ST_Equals()` | 一个几何体是否等于另一个 |  |  |
| `ST_ExteriorRing()` | 返回多边形的外环 |  |  |
| `ST_FrechetDistance()` | 一个几何体到另一个几何体的离散 Fréchet 距离 | 8.0.23 |  |
| `ST_GeoHash()` | 生成地理哈希值 |  |  |
| `ST_GeomCollFromText()`, `ST_GeometryCollectionFromText()`, `ST_GeomCollFromTxt()` | 从 WKT 返回几何集合 |  |  |
| `ST_GeomCollFromWKB()`, `ST_GeometryCollectionFromWKB()` | 从 WKB 返回几何集合 |  |  |
| `ST_GeometryN()` | 从几何集合中返回第 N 个几何体 |  |  |
| `ST_GeometryType()` | 返回几何类型的名称 |  |  |
| `ST_GeomFromGeoJSON()` | 从 GeoJSON 对象生成几何体 |  |  |
| `ST_GeomFromText()`, `ST_GeometryFromText()` | 从 WKT 返回几何体 |  |  |
| `ST_GeomFromWKB()`, `ST_GeometryFromWKB()` | 从 WKB 返回几何体 |  |  |
| `ST_HausdorffDistance()` | 一个几何体到另一个几何体的离散豪斯多夫距离 | 8.0.23 |  |
| `ST_InteriorRingN()` | 返回多边形的第 N 个内环 |  |  |
| `ST_Intersection()` | 返回两个几何体的点集交集 |  |  |
| `ST_Intersects()` | 判断一个几何体是否与另一个相交 |  |  |
| `ST_IsClosed()` | 判断几何体是否封闭且简单 |  |  |
| `ST_IsEmpty()` | 判断几何体是否为空 |  |  |
| `ST_IsSimple()` | 判断几何体是否简单 |  |  |
| `ST_IsValid()` | 判断几何体是否有效 |  |  |
| `ST_LatFromGeoHash()` | 从 geohash 值返回纬度 |  |  |
| `ST_Latitude()` | 返回 Point 的纬度 | 8.0.12 |  |
| `ST_Length()` | 返回 LineString 的长度 |  |  |
| `ST_LineFromText()`, `ST_LineStringFromText()` | 从 WKT 构建 LineString |  |  |
| `ST_LineFromWKB()`, `ST_LineStringFromWKB()` | 从 WKB 构建 LineString |  |  |
| `ST_LineInterpolatePoint()` | 沿着 LineString 给定百分比的点 | 8.0.24 |  |
| `ST_LineInterpolatePoints()` | 沿着 LineString 给定百分比的点 | 8.0.24 |  |
| `ST_LongFromGeoHash()` | 从 geohash 值返回经度 |  |  |
| `ST_Longitude()` | 返回 Point 的经度 | 8.0.12 |  |
| `ST_MakeEnvelope()` | 两点围成的矩形 |  |  |
| `ST_MLineFromText()`, `ST_MultiLineStringFromText()` | 从 WKT 构建 MultiLineString |  |  |
| `ST_MLineFromWKB()`, `ST_MultiLineStringFromWKB()` | 从 WKB 构建 MultiLineString |  |  |
| `ST_MPointFromText()`, `ST_MultiPointFromText()` | 从 WKT 构建 MultiPoint |  |  |
| `ST_MPointFromWKB()`, `ST_MultiPointFromWKB()` | 从 WKB 构建 MultiPoint |  |  |
| `ST_MPolyFromText()`, `ST_MultiPolygonFromText()` | 从 WKT 构建 MultiPolygon |  |  |
| `ST_MPolyFromWKB()`, `ST_MultiPolygonFromWKB()` | 从 WKB 构建 MultiPolygon |  |  |
| `ST_NumGeometries()` | 返回几何集合中的几何图形数量 |  |  |
| `ST_NumInteriorRing()`, `ST_NumInteriorRings()` | 返回多边形中的内部环数量 |  |  |
| `ST_NumPoints()` | 返回线串中的点数 |  |  |
| `ST_Overlaps()` | 一个几何图形是否与另一个重叠 |  |  |
| `ST_PointAtDistance()` | 沿着线串给定距离的点 | 8.0.24 |  |
| `ST_PointFromGeoHash()` | 将 geohash 值转换为 POINT 值 |  |  |
| `ST_PointFromText()` | 从 WKT 构造点 |  |  |
| `ST_PointFromWKB()` | 从 WKB 构造点 |  |  |
| `ST_PointN()` | 返回线串中第 N 个点 |  |  |
| `ST_PolyFromText()`, `ST_PolygonFromText()` | 从 WKT 构造多边形 |  |  |
| `ST_PolyFromWKB()`, `ST_PolygonFromWKB()` | 从 WKB 构造多边形 |  |  |
| `ST_Simplify()` | 返回简化后的几何图形 |  |  |
| `ST_SRID()` | 返回几何图形的空间参考系统 ID |  |  |
| `ST_StartPoint()` | 线串的起始点 |  |  |
| `ST_SwapXY()` | 返回 X/Y 坐标交换的参数 |  |  |
| `ST_SymDifference()` | 返回两个几何图形的点集对称差 |  |  |
| `ST_Touches()` | 一个几何图形是否与另一个相接触 |  |  |
| `ST_Transform()` | 转换几何图形的坐标 | 8.0.13 |  |
| `ST_Union()` | 返回两个几何图形的点集并集 |  |  |
| `ST_Validate()` | 返回经过验证的几何图形 |  |  |
| `ST_Within()` | 一个几何图形是否在另一个内部 |  |  |
| `ST_X()` | 返回点的 X 坐标 |  |  |
| `ST_Y()` | 返回点的 Y 坐标 |  |  |
| `STATEMENT_DIGEST()` | 计算语句摘要哈希值 |  |  |
| `STATEMENT_DIGEST_TEXT()` | 计算规范化语句摘要 |  |  |
| `STD()` | 返回总体标准偏差 |  |  |
| `STDDEV()` | 返回总体标准偏差 |  |  |
| `STDDEV_POP()` | 返回总体标准偏差 |  |  |
| `STDDEV_SAMP()` | 返回样本标准偏差 |  |  |
| `STR_TO_DATE()` | 将字符串转换为日期 |  |  |
| `STRCMP()` | 比较两个字符串 |  |  |
| `SUBDATE()` | 在使用三个参数调用时，DATE_SUB()的同义词 |  |  |
| `SUBSTR()` | 返回指定的子字符串 |  |  |
| `SUBSTRING()` | 返回指定的子字符串 |  |  |
| `SUBSTRING_INDEX()` | 返回指定分隔符出现次数之前的字符串子串 |  |  |
| `SUBTIME()` | 减去时间 |  |  |
| `SUM()` | 返回总和 |  |  |
| `SYSDATE()` | 返回函数执行时的时间 |  |  |
| `SYSTEM_USER()` | USER()的同义词 |  |  |
| `TAN()` | 返回参数的正切 |  |  |
| `TIME()` | 提取传递表达式的时间部分 |  |  |
| `TIME_FORMAT()` | 格式化为时间 |  |  |
| `TIME_TO_SEC()` | 返回转换为秒的参数 |  |  |
| `TIMEDIFF()` | 减去时间 |  |  |
| `TIMESTAMP()` | 使用单个参数，此函数返回日期或日期时间表达式；使用两个参数，返回参数的总和 |  |  |
| `TIMESTAMPADD()` | 将间隔添加到日期时间表达式 |  |  |
| `TIMESTAMPDIFF()` | 返回两个日期时间表达式的差异，使用指定的单位 |  |  |
| `TO_BASE64()` | 返回转换为 base-64 字符串的参数 |  |  |
| `TO_DAYS()` | 返回转换为天数的日期参数 |  |  |
| `TO_SECONDS()` | 返回自公元 0 年以来的秒数转换为日期或日期时间参数 |  |  |
| `TRIM()` | 移除前导和尾随空格 |  |  |
| `TRUNCATE()` | 截断到指定的小数位数 |  |  |
| `UCASE()` | UPPER()的同义词 |  |  |
| `UNCOMPRESS()` | 解压缩压缩的字符串 |  |  |
| `UNCOMPRESSED_LENGTH()` | 返回压缩前字符串的长度 |  |  |
| `UNHEX()` | 返回包含数字的十六进制表示的字符串 |  |  |
| `UNIX_TIMESTAMP()` | 返回 Unix 时间戳 |  |  |
| `UpdateXML()` | 返回替换的 XML 片段 |  |  |
| `UPPER()` | 转换为大写 |  |  |
| `USER()` | 客户端提供的用户名和主机名 |  |  |
| `UTC_DATE()` | 返回当前的 UTC 日期 |  |  |
| `UTC_TIME()` | 返回当前的 UTC 时间 |  |  |
| `UTC_TIMESTAMP()` | 返回当前的 UTC 日期和时间 |  |  |
| `UUID()` | 返回通用唯一标识符（UUID） |  |  |
| `UUID_SHORT()` | 返回整数值通用标识符 |  |  |
| `UUID_TO_BIN()` | 将字符串 UUID 转换为二进制 |  |  |
| `VALIDATE_PASSWORD_STRENGTH()` | 确定密码强度 |  |  |
| `VALUES()` | 定义在 INSERT 期间要使用的值 |  |  |
| `VAR_POP()` | 返回总体标准方差 |  |  |
| `VAR_SAMP()` | 返回样本方差 |  |  |
| `VARIANCE()` | 返回总体标准方差 |  |  |
| `VERSION()` | 返回指示 MySQL 服务器版本的字符串 |  |  |
| `WAIT_FOR_EXECUTED_GTID_SET()` | 等待副本上执行给定的 GTIDs。 |  |  |
| `WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()` | 使用 `WAIT_FOR_EXECUTED_GTID_SET()`。 |  | 8.0.18 |
| `WEEK()` | 返回周数 |  |  |
| `WEEKDAY()` | 返回星期索引 |  |  |
| `WEEKOFYEAR()` | 返回日期的日历周数（1-53） |  |  |
| `WEIGHT_STRING()` | 返回字符串的权重字符串 |  |  |
| `XOR` | 逻辑异或 |  |  |
| `YEAR()` | 返回年份 |  |  |
| `YEARWEEK()` | 返回年份和周数 |  |  |
| `&#124;` | 按位或 |  |  |
| `~` | 按位取反 |  |  |
| 名称 | 描述 | 引入版本 | 废弃版本 |
