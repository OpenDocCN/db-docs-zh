# 22.5.6 X 插件选项和变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/x-plugin-options-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-options-variables.html)

22.5.6.1 X 插件选项和变量参考

22.5.6.2 X 插件选项和系统变量

22.5.6.3 X 插件状态变量

本节描述了配置 X 插件的命令选项和系统变量，以及用于监控目的的状态变量。如果在启动时指定的配置值不正确，X 插件可能无法正确初始化，服务器也无法加载它。在这种情况下，服务器还可能因为无法识别其他 X 插件设置而产生错误消息。
