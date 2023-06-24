# Linux 速查

## 文件权限 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Linux文件权限.jpg)


# htop解析

htop 是一个类似于 top 命令的工具，用于监视系统资源的使用情况。下面是各列的含义：

- `PID`: 进程 ID。
- `USER`: 进程拥有者的用户名。
- `PR`: 进程的优先级别。PR 值越小表示进程优先级越高。
- `NI`: 进程的 nice 值。nice 值是一个影响进程调度优先级的参数。nice 值越低表示进程的优先级越高，默认值为 0。
- `VIRT`: 进程占用的虚拟内存大小（单位为 KB）。
- `RES`: 进程占用的物理内存大小（单位为 KB）。
- `SHR`: 进程占用的共享内存大小（单位为 KB）。
- `S (%CPU)`: 进程状态和 CPU 占用率。S 列显示进程的状态，其中 D 表示不可中断的休眠状态（通常是因为 I/O 操作），R 表示运行状态，S 表示睡眠状态，T 表示被跟踪或停止状- 态，Z 表示僵尸状态。%CPU 列显示进程在最近一次刷新间使用 CPU 的百分比，即 CPU 占用率。
- `MEM (%MEM)`: 进程占用的内存大小和内存使用率。MEM 显示进程占用的物理内存大小（单位为 KB），%MEM 显示进程占用的物理内存大小在系统总内存中的比例。
- `TIME+`: 进程使用 CPU 的累计时间，包括用户态和内核态。格式为 HH:MM:SS。
- `Command`: 进程的命令行。 


这些列中有一些是默认情况下不显示的，可以通过按下 F2 键打开设置菜单，进入"Columns"选项卡，选择需要显示的列并保存设置即可。





## 参考资料
> - [Linux Tools Quick Tutorial](https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html)
> - [命令行的艺术GitHub](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md)
