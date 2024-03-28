# 17.8.8 配置自旋锁轮询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-performance-spin_lock_polling.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-spin_lock_polling.html)

`InnoDB` 互斥锁和读写锁通常保留在短时间间隔内。在多核系统上，一个线程在睡眠之前连续检查是否可以在一段时间内获取互斥锁或读写锁可能更有效。如果在此期间互斥锁或读写锁变为可用，线程可以立即继续，在同一时间片内。然而，多个线程频繁轮询共享对象（如互斥锁或读写锁）可能导致“缓存乒乓”，这会导致处理器使彼此的缓存部分失效。`InnoDB`通过强制在轮询活动之间引入随机延迟来最小化此问题。随机延迟实现为自旋等待循环。

自旋等待循环的持续时间取决于循环中发生的 PAUSE 指令数量。该数字是通过随机选择一个整数，范围从 0 到`innodb_spin_wait_delay`值，然后将该值乘以 50 来生成的。（在 MySQL 8.0.16 之前，乘数值 50 是硬编码的，在此之后可配置。）例如，对于`innodb_spin_wait_delay`设置为 6，会随机选择以下范围内的一个整数：

```sql
{0,1,2,3,4,5}
```

所选整数乘以 50，得到六个可能的 PAUSE 指令值之一：

```sql
{0,50,100,150,200,250}
```

对于这组值，250 是自旋等待循环中可能发生的 PAUSE 指令的最大数量。当`innodb_spin_wait_delay`设置为 5 时，会得到一组五个可能的值`{0,50,100,150,200}`，其中 200 是 PAUSE 指令的最大数量，依此类推。这样，`innodb_spin_wait_delay`设置控制着自旋锁轮询之间的最大延迟。

在所有处理器核心共享快速缓存内存的系统上，您可以通过设置`innodb_spin_wait_delay=0`来减少最大延迟或完全禁用忙等待循环。在具有多个处理器芯片的系统上，缓存失效的影响可能更显著，您可能需要增加最大延迟。

在 100MHz 奔腾时代，一个`innodb_spin_wait_delay`单位被校准为相当于一微秒。那个时间等价性并不成立，但`PAUSE`指令持续时间相对于其他 CPU 指令的处理器周期保持相对稳定，直到 Skylake 处理器的引入，这些处理器具有相对较长的`PAUSE`指令。`innodb_spin_wait_pause_multiplier`变量在 MySQL 8.0.16 中引入，以提供一种解决`PAUSE`指令持续时间差异的方法。

`innodb_spin_wait_pause_multiplier`变量控制`PAUSE`指令值的大小。例如，假设`innodb_spin_wait_delay`设置为 6，将`innodb_spin_wait_pause_multiplier`值从 50（默认值和以前硬编码值）减少到 5 会生成一组较小的`PAUSE`指令值：

```sql
{0,5,10,15,20,25}
```

增加或减少`PAUSE`指令值的能力允许对不同处理器架构进行微调`InnoDB`。较小的`PAUSE`指令值适用于具有相对较长`PAUSE`指令的处理器架构，例如。

`innodb_spin_wait_delay`和`innodb_spin_wait_pause_multiplier`变量是动态的。它们可以在 MySQL 选项文件中指定，或者使用`SET GLOBAL`语句在运行时进行修改。在运行时修改变量需要足够设置全局系统变量的权限。参见第 7.1.9.1 节，“系统变量权限”。
