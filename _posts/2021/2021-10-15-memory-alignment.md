---
layout:   post
title:   "内存对齐"
subtitle:  "memory alignment"
date:    2021-10-15 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 内存对齐
---

### 术语定义

-  naturally aligned

参考自http://books.gigatux.nl/mirror/kerneldevelopment/0672327201/ch19lev1sec3.html

>  A variable is *naturally aligned* if it exists at a memory address that is a multiple of its size. For example, a 32-bit type is naturally aligned if it is located in memory at an address that is a multiple of four (that is, its lowest two bits are zero). Thus, a data type with size 2*n* bytes must have an address with the *n* least significant bits set to zero.

当访问一块N字节的内存变量时，若访问的内存地址能被N整除，则称该内存地址是 naturally aligned。



- data alignment

参考自https://en.m.wikipedia.org/wiki/Data_structure_alignment

> Data alignment is the aligning of elements according to their natural alignment. To ensure natural alignment, it may be necessary to insert some padding between structure elements or after the last element of a structure. For example, on a 32-bit machine, a data structure containing a 16-bit value followed by a 32-bit value could have 16 bits of padding between the 16-bit value and the 32-bit value to align the 32-bit value on a 32-bit boundary. Alternatively, one can pack the structure, omitting the padding, which may lead to slower access, but uses three quarters as much memory.

举例来说，在linux x86平台下，定义了如下的结构体：

```
struct mystruct {
    char c; // 1 byte
    short s; // 2 byte
    int i; // 4 byte
};
```

按照上述*naturally aligned*的定义，如果要访问占用1字节的c的内存地址，无论如何都是naturally aligned，但此时访问占用2字节的s却无法满足naturally aligned，解决的办法就是在c后面插入1个字节(也叫padding)，使得c和s一共占用4字节，而对于i来说，刚好也满足naturally aligned，这样一来结构体的所有变量内存地址就都满足naturally aligned，我们称这样的结构体为data alignment。



- memory access granularity  vs  memory access boundary

参考自https://stackoverflow.com/questions/16620226/double-byte-memory-access-granularity

> *Memory access granularity* is the number of bytes it accesses at a time, and a *memory access boundary* is where each of these groups of bytes begins.

cpu需要从内存加载数据，那么cpu访问一次内存获取的数据大小，称之为memory access granularity，单位是byte，比如68000下是2字节，68030 和PowerPC® 601是4字节。



### 什么是内存对齐

有了上述*naturally aligned*和*data alignment*的定义，便不难理解什么是内存对齐，这里举出一个反例：

```
    char dog[10];
    char *p = &dog[1];
    unsigned long l = *(unsigned long *)p;
```

上述代码里，指向char的指针被强转为unsigned long型，此时访问变量l，便有可能会造成内存不对齐的问题。

### 为什么需要内存对齐

一般的，编程中大多数内存不对齐的问题都会在编译器这一层解决了，但是出于好奇，我们也会想知道为什么要做内存对齐，或者说内存对齐带来了哪些好处？

- 访问速度

假设我们现在用的是PowerPC® 601，其memory access granularity是4字节，（下图来自https://developer.ibm.com/technologies/systems/articles/pa-dalign/)是分别从0x0和0x1读取4字节的示意图：

![](/img/in-post/zhouhaijun-pic/memory_alig_1.jpeg)

从图中可以看出，当从地址0x0读取4字节只需要一次读取即可，但是从地址0x1读取4字节，就需要2次，具体过程如下图：

![](/img/in-post/zhouhaijun-pic/unalignedAccess.jpeg)

- [ ] 第一步从0x0读取4字节，获取字节0x1~0x3；
- [ ] 第二步从0x4读取4字节，获取字节0x4；
- [ ] 两次读取后的数据再做一次聚合；

由此得知，内存不对齐会导致访问内存的频次变高，一是降低了访问速度，另外一个是增加了总线带宽的消耗。

- 原子性

当前的很多处理器都提供了一些原子操作的指令，在使用这些指令去访问内存时，如果提供的地址没有满足内存对齐，会导致的访问的内存变成多次，进而原子的语义就会被破坏。

- 兼容性问题

在x86的平台下我们借助pragma指令可以强制编译器修改默认对齐字节数，是因为x86的处理器可以访问不对齐的内存，但在其他的处理器上，例如68000 下cpu访问不对齐的地址时会抛出异常，在Sun SPARC下访问未对齐的内存程序会crash。





### 结构体内存对齐规则

- 对齐规则

N=min(结构体最宽的成员大小，指定编译器对齐大小)，注意制定编译器对齐大小只能是2的幂，比如1、2、4字节，实际编程中尽量不要使用pragma来制定编译器对齐大小，可能会导致不同平台下的兼容性问题：

- [ ] 结构体变量的起始地址能够被N整除；
- [ ] 结构体每个成员相对于起始地址的偏移能够被其自身大小整除，如果不能则在前一个成员后面补充字节；
- [ ] 结构体总体大小能够被N整除，如不能则在后面补充字节；



- 实例

定义如下结构体，在linux  x86平台下的内存布局为：

```
struct mystruct {
    bool a;
    // 为确保访问b的内存对齐，此处插入1字节
    short b;
    // 为确保访问c时内存对齐，此处插入4字节
    long long c; // 成员变量最宽字节数，8字节
    bool d;
    // 为确保数组元素的下一个结构体首地址内存对齐，此处插入7字节
};
```

### 参考资料



内存对齐

- https://developer.ibm.com/technologies/systems/articles/pa-dalign/
- https://en.m.wikipedia.org/wiki/Data_structure_alignment



为什么memory access boundary is power of 2

- https://stackoverflow.com/questions/64415023/why-cpu-accesses-aligned-memory
- https://stackoverflow.com/questions/3655926/why-does-cpu-access-memory-on-a-word-boundary
