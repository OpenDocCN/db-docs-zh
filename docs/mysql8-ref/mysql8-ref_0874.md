# 14.22 内部函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/internal-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/internal-functions.html)

**表格 14.32 内部函数**

| 名称 | 描述 | 引入版本 |
| --- | --- | --- |
| `CAN_ACCESS_COLUMN()` | 仅供内部使用 |  |
| `CAN_ACCESS_DATABASE()` | 仅供内部使用 |  |
| `CAN_ACCESS_TABLE()` | 仅供内部使用 |  |
| `CAN_ACCESS_USER()` | 仅供内部使用 | 8.0.22 |
| `CAN_ACCESS_VIEW()` | 仅供内部使用 |  |
| `GET_DD_COLUMN_PRIVILEGES()` | 仅供内部使用 |  |
| `GET_DD_CREATE_OPTIONS()` | 仅供内部使用 |  |
| `GET_DD_INDEX_SUB_PART_LENGTH()` | 仅供内部使用 |  |
| `INTERNAL_AUTO_INCREMENT()` | 仅供内部使用 |  |
| `INTERNAL_AVG_ROW_LENGTH()` | 仅供内部使用 |  |
| `INTERNAL_CHECK_TIME()` | 仅供内部使用 |  |
| `INTERNAL_CHECKSUM()` | 仅供内部使用 |  |
| `INTERNAL_DATA_FREE()` | 仅供内部使用 |  |
| `INTERNAL_DATA_LENGTH()` | 仅供内部使用 |  |
| `INTERNAL_DD_CHAR_LENGTH()` | 仅供内部使用 |  |
| `INTERNAL_GET_COMMENT_OR_ERROR()` | 仅供内部使用 |  |
| `INTERNAL_GET_ENABLED_ROLE_JSON()` | 仅供内部使用 | 8.0.19 |
| `INTERNAL_GET_HOSTNAME()` | 仅供内部使用 | 8.0.19 |
| `INTERNAL_GET_USERNAME()` | 仅供内部使用 | 8.0.19 |
| `INTERNAL_GET_VIEW_WARNING_OR_ERROR()` | 仅供内部使用 |  |
| `INTERNAL_INDEX_COLUMN_CARDINALITY()` | 仅供内部使用 |  |
| `INTERNAL_INDEX_LENGTH()` | 仅供内部使用 |  |
| `INTERNAL_IS_ENABLED_ROLE()` | 仅供内部使用 | 8.0.19 |
| `INTERNAL_IS_MANDATORY_ROLE()` | 仅供内部使用 | 8.0.19 |
| `INTERNAL_KEYS_DISABLED()` | 仅供内部使用 |  |
| `INTERNAL_MAX_DATA_LENGTH()` | 仅供内部使用 |  |
| `INTERNAL_TABLE_ROWS()` | 仅供内部使用 |  |
| `INTERNAL_UPDATE_TIME()` | 仅供内部使用 |  |
| 名称 | 描述 | 引入版本 |

本节列出的函数仅供服务器内部使用。用户尝试调用它们会导致错误。

+   `CAN_ACCESS_COLUMN(*`ARGS`*)`

+   `CAN_ACCESS_DATABASE(*`ARGS`*)`

+   `CAN_ACCESS_TABLE(*`ARGS`*)`

+   `CAN_ACCESS_USER(*`ARGS`*)`

+   `CAN_ACCESS_VIEW(*`ARGS`*)`

+   `GET_DD_COLUMN_PRIVILEGES(*`ARGS`*)`

+   `GET_DD_CREATE_OPTIONS(*`ARGS`*)`

+   `GET_DD_INDEX_SUB_PART_LENGTH(*`ARGS`*)`

+   `INTERNAL_AUTO_INCREMENT(*`ARGS`*)`

+   `INTERNAL_AVG_ROW_LENGTH(*`ARGS`*)`

+   `INTERNAL_CHECK_TIME(*`ARGS`*)`

+   `INTERNAL_CHECKSUM(*`ARGS`*)`

+   `INTERNAL_DATA_FREE(*`ARGS`*)`

+   `INTERNAL_DATA_LENGTH(*`ARGS`*)`

+   `INTERNAL_DD_CHAR_LENGTH(*`ARGS`*)`

+   `INTERNAL_GET_COMMENT_OR_ERROR(*`ARGS`*)`

+   `INTERNAL_GET_ENABLED_ROLE_JSON(*`ARGS`*)`

+   `INTERNAL_GET_HOSTNAME(*`ARGS`*)`

+   `INTERNAL_GET_USERNAME(*`ARGS`*)`

+   `INTERNAL_GET_VIEW_WARNING_OR_ERROR(*`ARGS`*)`

+   `INTERNAL_INDEX_COLUMN_CARDINALITY(*`ARGS`*)`

+   `INTERNAL_INDEX_LENGTH(*`ARGS`*)`

+   `INTERNAL_IS_ENABLED_ROLE(*`ARGS`*)`

+   `INTERNAL_IS_MANDATORY_ROLE(*`ARGS`*)`

+   `INTERNAL_KEYS_DISABLED(*`ARGS`*)`

+   `INTERNAL_MAX_DATA_LENGTH(*`ARGS`*)`

+   `INTERNAL_TABLE_ROWS(*`ARGS`*)`

+   `INTERNAL_UPDATE_TIME(*`ARGS`*)`

+   `IS_VISIBLE_DD_OBJECT(*`ARGS`*)`
