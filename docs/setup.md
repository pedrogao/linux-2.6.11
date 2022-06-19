# setup 启动

对应 [init/main.c](../init/main.c) 中的内容，PC通电后，从 BIOS、bootloader 一路到 kernel，
kernel 中的启动函数 `start_kernel` 函数会做一系列的初始化工作，包括：
- 调度初始化
- trap 初始化
- 软、硬中断初始化
- 资源初始化
    - 第一个用户态进程，1号进程