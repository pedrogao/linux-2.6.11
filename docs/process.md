# 进程

Linux 使用轻量级进程，进程之间可以共享地址控件、文件描述符等资源；进程、线程均由 task_struct 来描述，即任务。

task_struct 被称为进程描述符，每一个进程都有唯一的Id标识，被称为 process id，CPU通过双向链表来关联每个 task_struct。

注意：对于父子进程，real_parent是其真正的父进程，如果没有父进程，则指向1号init进程；parent指向当前父进程，如 ptrace时，
会指向 ptrace 进程。

在 x86 平台下，进程相关的处理代码在 arch/x86-64/kernel/process.c 文件中。

