---
layout:   post
title:   "性能调优(一)系统负载"
subtitle:  ""
date:    2021-09-16 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 性能
---

### 系统负载

- 什么是系统负载？

在Linux系统下，我们可以通过top或者uptime命令输出，看到如下的字符显示：

> 06:02:05 up 12 days, 13:35,  5 users,  load average: 2.33, 1.73, 1.58

其中load average代表的就是系统负载：2.33, 1.73, 1.58，分表代表过去的1分钟、5分钟、15分钟的系统负载。

那么这个数字代表什么意思呢？通过man  uptime可以看到对系统平均负载的一段解释：

> System load averages is the average number of processes that are either in a runnable or uninterruptable state.  A process in a runnable state is either using the CPU or waiting to use the CPU.  A process in  uninter-
>
> ruptable  state  is waiting for some I/O access, eg waiting for disk.  The averages are taken over the three time intervals.  Load averages are not normalized for the number of CPUs in a system, so a load average of 1
>
> means a single CPU system is loaded all the time while on a 4 CPU system it means it was idle 75% of the time.

也就是说，系统负载代表的是单位时间内的活跃进程数量，所谓活跃进程，也就是状态处于runnable或者uninterruptable的进程：我们把正在使用cpu或者等待使用cpu的进程称为runnable，这个地方分为两部分，一部分是正在运行cpu指令的进程，另一部分是指存在操作系统调度队列的进程，等待被调度的，涉及到上下文切换的进程；而uninterruptalbe指的是一些正在等待IO处理的进程数，比如写磁盘等等。



- 出现系统过载时如何进一步排查？

那么系统平均负载这个数字背后的含义是什么？或者说这个指标数据对我们有什么帮助？

一般地，我们认为系统的平均负载与cpu核的数量一致，理想的情况任意时间内有且仅有一个线程占用一个cpu，换言之，当系统平均负载大于cpu核数时，说明我们的系统已经过载了，而在实际的生产环境里边，可能当平均负载超过80%，就得去排查一下问题了。



通过上面的描述，我们知道系统过载一般是由三种原因导致的：第一种就是cpu在玩命的执行指令，这种被称为cpu密集型；第二种就是系统进程数量过多，导致大量进程在等待cpu执行，也就是处于cpu待调度的调度队列里；第三种就是有大量的进程在等待IO，这种被称为IO密集型。

那么当系统出现过载时，我们应该如何去排查问题呢？或者说如何找到具体的原因呢？

这时我们需要用到几种工具：vmstat、iostat、sar、mpstat、pidstat、perf、strace。vmstat用于统计进程、内存、swap、io、上下文切换、cpu使用率，通过它可以观测到整个系统的整体情况；iostat用于统计磁盘使用情况，可以观测到磁盘IO负载；sar是用于统计网络IO的负载；mpstat用于查看系统每个CPU的使用率;pidstat是按照进程来统计内存、磁盘IO、CPU使用率、上下文切换；使用perf可以追踪系统中的一些热点函数，而strace可以查看进程的函数调用情况；



按照我的使用习惯，一般通过top或者uptime确认系统是否已经过载，如果过载，那么会使用vmstat整体观测一下进程、内存、cpu使用情况，分别使用iostat和sar分别查看磁盘和网络IO使用率，确认是否是IO瓶颈导致的负载过高，之后再结合mpstat以及pidstat进一步查看具体是哪个或者哪些进程导致的系统过载，最后使用perf可以查看到系统一些热点函数，使用strace可以查看一个进程的函数调用情况，通过以上几步基本上就能定位出原因或者说找到一些线索。



### cpu使用率

- cpu指标数据涵义

在使用上面的几种工具(vmstat/pidstat/mpstat)采样CPU使用率时，有几个关键性的指标数据：

usr：应用层程序消耗的cpu使用率；

sys：内核层程序消耗的cpu使用率；

idle：表示cpu处于空闲状态；

iowait：程序在等待一些IO，但cpu此时也处于空闲状态；



- 系统负载与cpu使用率区别

系统负载是对系统整体状况的综合判断，而cpu使用率则是单纯的统计cpu的繁忙程度，两者有一定的关联，但其实还有些不一样，主要体现在以下几个点上：

🔲当系统负载过高时，如果是由于cpu密集型导致的，那么这种情况下和cpu使用率是一致的；

🔲当然系统过载也有可能是由于大量的进程在等待io导致的，比如磁盘或网络，那么这种情况下cpu反而很空闲，这种情况下其实是我们的io出现率瓶颈，比如说磁盘速度已经饱和或者网络带宽已经达到上限；



### 延伸阅读

[Linux Load Averages: Solving the Mystery](http://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html)

[Context Switch Definition](http://www.linfo.org/context_switch.html)

[System Call Definition](http://www.linfo.org/system_call.html)

[What exactly is "iowait"?](https://blog.pregos.info/wp-content/uploads/2010/09/iowait.txt)

