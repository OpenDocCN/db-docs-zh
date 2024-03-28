> 原文：[`dev.mysql.com/doc/refman/8.0/en/ddl-rewriter-options.html`](https://dev.mysql.com/doc/refman/8.0/en/ddl-rewriter-options.html)

#### 7.6.5.2 ddl_rewriter 插件选项

本节描述了控制`ddl_rewriter`插件操作的命令选项。如果在启动时指定的值不正确，则`ddl_rewriter`插件可能无法正确初始化，服务器也不会加载它。

要控制`ddl_rewriter`插件的激活，请使用此选项：

+   [`--ddl-rewriter[=*`value`*]`](ddl-rewriter-options.html#option_mysqld_ddl-rewriter)

    | 命令行格式 | `--ddl-rewriter[=value]` |
    | --- | --- |
    | 引入版本 | 8.0.16 |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``FORCE``FORCE_PLUS_PERMANENT` |

    此选项控制服务器在启动时如何加载`ddl_rewriter`插件。仅当插件先前已通过`INSTALL PLUGIN`注册或通过`--plugin-load`或`--plugin-load-add`加载时才可用。参见 Section 7.6.5.1, “Installing or Uninstalling ddl_rewriter”。

    选项值应为插件加载选项中可用的值之一，如 Section 7.6.1, “Installing and Uninstalling Plugins”中所述。例如，`--ddl-rewriter=OFF`可在服务器启动时禁用插件。
