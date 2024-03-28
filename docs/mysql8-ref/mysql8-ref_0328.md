# 7.7.2 获取有关可加载函数的信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/obtaining-loadable-function-information.html`](https://dev.mysql.com/doc/refman/8.0/en/obtaining-loadable-function-information.html)

Performance Schema `user_defined_functions` 表包含当前安装的可加载函数的信息：

```sql
SELECT * FROM performance_schema.user_defined_functions;
```

`mysql.func` 系统表还列出了已安装的可加载函数，但仅列出使用 `CREATE FUNCTION` 安装的函数。`user_defined_functions` 表列出了使用 `CREATE FUNCTION` 安装的可加载函数，以及由组件或插件自动安装的可加载函数。这种差异使得 `user_defined_functions` 比 `mysql.func` 更适合检查已安装的可加载函数。参见 Section 29.12.21.10, “用户定义函数表”。
