> [`dev.mysql.com/doc/refman/8.0/en/sys-ps-check-lost-instrumentation.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-ps-check-lost-instrumentation.html)

#### 30.4.3.23 ps_check_lost_instrumentation 视图

此视图返回有关丢失的性能模式仪器的信息，以指示性能模式是否无法监视所有运行时数据。

`ps_check_lost_instrumentation` 视图具有以下列：

+   `variable_name`

    指示丢失哪种类型仪器的性能模式状态变量名称。

+   `variable_value`

    丢失的仪器数量。
