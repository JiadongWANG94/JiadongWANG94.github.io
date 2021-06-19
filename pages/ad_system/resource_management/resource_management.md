---
layout: default
title: 系统资源管理
parent: 自动驾驶系统
---

# 系统资源管理
系统资源包括CPU计算资源、内存、磁盘、网络带宽等。对于一个拥有很多进程的自动驾驶系统来说，可以通过资源管理来为重要模块的进程分配更多的资源，避免次要的进程争夺资源。

## cgroup
### 能力

### 设置方式

## CPU亲和性
一个进程/线程的CPU亲和性（affinity）指的是这个进程/线程被调度到某个/某些特定CPU核心上的倾向性。现代CPU有多级缓存，其中一些缓存是与核心绑定的（如L1 Cache和L2 Cache），如果一个进程被频繁调度到不同的核心上执行，那这些缓存的命中率会降低，设置CPU亲和性有助于提高缓存命中率，从而提高性能。网上有很多介绍CPU亲和性的文章。

### 软亲和性和硬亲和性
在未经任何设置的情况下，内核的调度算法会倾向于将任务尽可能地往同一个内核上进行调度，这被称为软亲和性。

在设置了affinity之后，CPU会强制将进程调度到特定的内核上，被称为硬亲和性。

### 设置方式
可以通过命令行，也可以通过c接口设置。

**命令行方式：**
```
Usage: taskset [options] [mask | cpu-list] [pid|cmd [args...]]

Show or change the CPU affinity of a process.

Options:
 -a, --all-tasks         operate on all the tasks (threads) for a given pid
 -p, --pid               operate on existing given pid
 -c, --cpu-list          display and specify cpus in list format

The default behavior is to run a new command:
    taskset 03 sshd -b 1024
You can retrieve the mask of an existing task:
    taskset -p 700
Or set it:
    taskset -p 03 700
List format uses a comma-separated list instead of a mask:
    taskset -pc 0,3,7-11 700
Ranges in list format can take a stride argument:
    e.g. 0-31:2 is equivalent to mask 0x55555555
```

**c接口方式：**
```cpp
int sched_setaffinity(pid_t pid, size_t cpusetsize,
                      cpu_set_t mask);
int sched_getaffinity(pid_t pid, size_t cpusetsize,
                      cpu_set_t mask);
```


## 参考
1. [关于CPU亲和性，这篇讲得最全面 - 知乎](https://zhuanlan.zhihu.com/p/259217757)