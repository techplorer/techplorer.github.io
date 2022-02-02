### 为什么需要缓存

现如今大部分的cpu cache都是hierarchy的结构，一般cpu内部会有三级缓存：L1/L2/L3，当cpu访问一个内存地址时，会先从L1缓存中查找，如果没有命中则从L2查找，如果还是没有命中继续去L3查找，如果都没有命中则访问内存，访问的速度依次是L1>L2>L3>Memory。

![cpu cache](http://www.techplorer.cn/upload/2021/08/cpu%20cache-f2e79283e5f943019e7d7086ad84f7e7.png)

那么到底cpu访问cache和memory的速度差异有多少，这里以AIDA64 CPU为例(数据参考自https://www.overclockers.com/intel-core-i9-9900k-cpu-review-more-cores-speed-and-higher-price/)：

![cpu cache benchmark](http://www.techplorer.cn/upload/2021/08/cpu%20cache%20benchmark-b4b72da624ea4ab0b1f34f167587dbf8.png)

从上述图中可以看出，cpu访问的速度大约是：

L1=3 * L2=4 * L3=4 * Memeory

也就是说，cpu访问L1的速度约是内存的48倍，这是什么概念，好比说访问一次内存需要48s，那么cpu就有47s的时间在打盹。

总的来说，相比于cache而言，cpu访问内存实在太慢了，所以cache主要是解决cpu访问内存慢的问题。



### Cache miss/hit

- cache line

结合一张图讲解一下几个基础概念，下文图片来自(https://manybutfinite.com/post/intel-cpu-caches/)

![cache mapping](http://www.techplorer.cn/upload/2021/08/cache%20mapping-96f403daed464211b047267e6f4acdc5.png)

前文说过cpu直接访问内存会很慢，同时因为[locality of reference](https://en.wikipedia.org/wiki/Locality_of_reference)的缘故，每次cpu从内存加载一整块连续的数据块到缓存，而不是一个字节一个字节的加载，我们称这样的一个个的数据块为一个cache line。还是以上述图片为例，图中"Offset into cache line"一共占用6bit，表示的就是一个cache line的大小，即2^6=64bytes。

为便于理解，可以把cache理解为一个二维的表格，表格中的每个单元格对应一个内存块，也就是一个cache line，cpu根据内存地址查找缓存时，先通过"Set Index"拿到内存块所在的行，之后通过"24-bit tag"遍历所在的列，如果匹配成功，则表示cache hit，如果没有匹配成功，则表示cache miss。对于cache hit的情况，则可以根据offset找到缓存中具体的地址；如果是cache miss的情形，需要从内存加载数据到缓存，如果缓存满则触发缓存驱逐机制。





- 缓存加载(associativity)

数据从内存加载到缓存的过程，英文里有一个专业的术语叫associativity，主要有三种实现途径：

第一种是direct mapping，适用于cpu均匀访问内存的场景；

第二种是fully associative，缺点是每次查找缓存是需要对所有cache line做一次搜索，比较费时；

第三种是set associative，也是目前主流的一种设计；

这里有一篇文章是实例讲述缓存加载过程的(https://www.cs.swarthmore.edu/~kwebb/cs31/f18/memhierarchy/caching.html)

### 缓存一致性

- 缓存写(write policies)

cache hit的场景下，缓存写一般有两种策略：write-through和write-back。

write-through表示写缓存命中时，将缓存更新后的数据立即同步写入内存，缺点也是显而易见的，就是速度慢；

write-back则不会立即将缓存数据同步至内存，而是将更新后的缓存数据先标记为“脏数据”，当触发缓存驱逐机制，再将被驱逐的“脏数据”同步写入内存，最后再清理掉缓存“脏数据”。

cache miss的场景下，cpu同步数据到内存也有两种策略：write allocate和no-write allocate。

write allocate表示写内存之前，先将内存数据加载到缓存，后面的操作就和write-through一样，而write allocate则是直接写内存。

一般cpu会根据不同的场景选择不同的读写策略。

- 缓存驱逐(evict)

当从内存加载数据到缓存时，如果缓存满，则需要从缓存中删除cache line数据，这里就涉及删除策略的问题，目前用的比较多就是LRU(Least recently used)算法。

- cache coherence

上文讨论过缓存写的问题，那如果同时有多个核心写缓存会发生什么问题呢？也就是说当多个cpu核心同时对同一个cache line的数据做写操作，那这里问题就会变复杂一些，涉及到缓存数据同步的问题，英文里称这样的问题为cache coherency。解决这个问题就需要用到几种协议：snooping、directory、mesi。

目前用的比较多的是 MESI协议，简单介绍一下，MESI协议定义了几种状态：

M(Modified)：cache  line仅位于当前cpu核心的cache，且对应的内存的cache line数据已经过时，也就是内存数据处于stale状态；

E(Exclusive)：cache line仅位于当前cpu核心的cache，且与内存的cache line数据一致；

S(Shared)：cache line位于当前cpu核心的cache，且其他cpu核心也有相同的数据拷贝，且数据与内存的cache line数据一致；

I(Invalid)：cache line不在当前cpu核心的cache；



下面是状态机的转换图：

![cache coherence](http://www.techplorer.cn/upload/2021/08/cache%20coherence-ea89ea5b36394687a364f6a7869b2358.png)

这里有一个交互的动画，可以实际感受一下，例子就不在此赘述：https://www.scss.tcd.ie/Jeremy.Jones/vivio/caches/MESIHelp.htm



### 测试用例

最后，让我们做几组数据测试一下，我的电脑配置是：

- 🔲L1有32KB，2way；
- 🔲L2有4096KB, 2way；
- 🔲L3是16384KB，1way； 
- 🔲cache line是64byes；



- TEST CASE 1:给定一个二维数组，分别按行访问、按列访问、随机访问，测试三者的速度差异：

```
//rowMajor
int * arr = new int[N * N];
for (int i = 0;i < N; ++i) {
    for (int j = 0;j < N; ++j) {
        arr[i * N + j] += j;
    }
}

// columnMajor
int * arr = new int[N * N];
for (int i = 0;i < N; ++i) {
    for (int j = 0;j < N; ++j) {
        arr[j * N + i] += j;
    }
}

// random stride
int * arr = new int[N * N];
for (int i = 0;i < N; ++i) {
    for (int j = 0;j < N; ++j) {
        arr[j * N + rand() % N] += j;
    }
}
```

这是跑在我电脑上的一组benchmark数据，针对上述三种情况分别做了3组测试，内存中二维数组的长度分别是1024 * 1024, 2048 * 2048, 4096 * 4096：

```
------------------------------------------------------------
Benchmark                  Time             CPU   Iterations
------------------------------------------------------------
rowMajor/1024           2.77 ms         2.77 ms          199
rowMajor/2048           10.7 ms         10.7 ms           65
rowMajor/4096           47.6 ms         47.5 ms           12
columnMajor/1024        4.74 ms         4.74 ms          157
columnMajor/2048        22.7 ms         22.7 ms           30
columnMajor/4096         345 ms          345 ms            2
randomStride/1024       15.9 ms         15.9 ms           46
randomStride/2048        178 ms          178 ms            4
randomStride/4096       1175 ms         1171 ms            1
```

从测试数据可以看出，按行读取比按列读取要快2-3倍，随机读是最慢的，最差的情况下要比按行读慢20倍。





- TEST CASE 2:给定一个数组，每次按一定的步长访问第kth个地址

```
// array size : 512M
int N = 128 * 1024 * 1024;
int* arr = new int[N];

for (int i = 0;i < N; i += step) {
    arr[i] += 1;
}
```

跑出来的结果是：

```
---------------------------------------------------------
Benchmark               Time             CPU   Iterations
---------------------------------------------------------
accessKth/1        549098 us       549048 us            1
accessKth/2        309559 us       304412 us            2
accessKth/4        138416 us       137645 us            5
accessKth/8         71559 us        71555 us            8
accessKth/16        52139 us        52061 us           10
accessKth/32        45386 us        45382 us           12
accessKth/64        26680 us        26664 us           24
accessKth/128       14933 us        14902 us           42
accessKth/256        7555 us         7554 us           85
accessKth/512        4517 us         4517 us          128
accessKth/1024       2571 us         2571 us          216
accessKth/2048       1183 us         1180 us          593
accessKth/4096        630 us          629 us         1088
accessKth/8192        340 us          339 us         2126
```

测试结果发现，当cpu按步长[1, 2, 4, 8,16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192]去访问内存数据时，每次速度会提升2倍左右。

因为我的机器上cache line是64bytes，cpu每次从内存加载数据都是以cache line为最小单位的，而一个int变量占4字节，所以[1,2,4,,8]访问内存的次数是相同的，其差异在于cpu运算的次数，每次少一半，所以速度上会有2倍的提升；

对于[8,16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192]来说，速度的差异体现在两个方面：一方面是cpu访问内存的次数每次少一半，两外一方面是cpu运算次数每次也少一半，所以步长越长，速度上提升2倍多一点。



- TEST CASE 3:开一个线程读写一个变量4k万次，开四个线程分别分别读写同一个变量1k万次，开4个线程分别读写不同的变量1k万次：

```
// Simple function for incrememnting an atomic int
void work(std::atomic<int>& a) {
  for (int i = 0; i < 10000000; i++) {
    a++;
  }
}

// 一个线程读写一个变量4k万次
void single_thread() {
  std::atomic<int> a;
  a = 0;

  work(a);
  work(a);
  work(a);
  work(a);
}

// 四个线程分别分别读写同一个变量1k万次
void same_var() {
  std::atomic<int> a;
  a = 0;

  // Create four threads and use a lambda to launch work
  std::thread t1([&]() { work(a); });
  std::thread t2([&]() { work(a); });
  std::thread t3([&]() { work(a); });
  std::thread t4([&]() { work(a); });

  // Join the threads
  t1.join();
  t2.join();
  t3.join();
  t4.join();
}

// 4个线程分别读写不同的变量1k万次
void diff_var() {
  std::atomic<int> a{0}; // 0x7ffce7db3b9c
  std::atomic<int> b{0}; // 0x7ffce7db3b98
  std::atomic<int> c{0}; // 0x7ffce7db3b94
  std::atomic<int> d{0}; // 0x7ffce7db3b90

  // Creat four threads and use lambda to launch work
  std::thread t1([&]() { work(a); });
  std::thread t2([&]() { work(b); });
  std::thread t3([&]() { work(c); });
  std::thread t4([&]() { work(d); });

  // Join the threads
  t1.join();
  t2.join();
  t3.join();
  t4.join();
}
```

跑出来的数据是这样：

```
Run on (2 X 2595.12 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x2)
  L1 Instruction 32 KiB (x2)
  L2 Unified 4096 KiB (x2)
  L3 Unified 16384 KiB (x1)
Load Average: 0.03, 0.15, 0.23
------------------------------------------------------------------
Benchmark                        Time             CPU   Iterations
------------------------------------------------------------------
singleThread                   269 ms          268 ms            3
directSharing/real_time        275 ms        0.149 ms            3
falseSharing/real_time         277 ms        0.166 ms            3
```

单线程读写同一个变量4k万次约是269ms，之后我们开4个线程同时读写同一个变量1k万次，发现两者速度是一样的，在多核环境下，对于同一个内存地址，不同cpu核心里是相同cache line拷贝，当涉及多核写时，涉及到缓存数据一致性的问题，也就是上述说的mesi协议要做的事，那么cpu内部会消耗一部分的时间来同步数据。

再之后，我们开启4个线程分别对4个不同变量地址做1k万次的读写操作，惊奇地发现速度也是差不多，约是277ms，这就是false sharing的问题，虽然4个不同的变量地址不一样，虽然它们之间相差4个字节，但在内存里是连续排列的，而我的电脑的cache line是64字节，所以正好是同一个cache line(我在代码注释里添加里对应的内存地址)，所以也会出现大量cpu缓存冲突的问题。

对于false sharing的问题，代码稍微改一下，性能就会得到提升：

```
// We can align the struct to 64 bytes
// Now each struct will be on a different cache line
struct alignas(64) AlignedType {
  AlignedType() { val = 0; }
  std::atomic<int> val;
};

// No more invalidations, so our code should be roughly the same as the
void diff_line() {
  AlignedType a{}; // 0x7ffce7db3b40
  AlignedType b{}; // 0x7ffce7db3b00
  AlignedType c{}; // 0x7ffce7db3ac0
  AlignedType d{}; // 0x7ffce7db3a80

  // Launch the four threads now using our aligned data
  std::thread t1([&]() { work(a.val); });
  std::thread t2([&]() { work(b.val); });
  std::thread t3([&]() { work(c.val); });
  std::thread t4([&]() { work(d.val); });

  // Join the threads
  t1.join();
  t2.join();
  t3.join();
  t4.join();
}
```

做一下性能测试：

```
Running ./cache_example_04
Run on (2 X 2595.12 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x2)
  L1 Instruction 32 KiB (x2)
  L2 Unified 4096 KiB (x2)
  L3 Unified 16384 KiB (x1)
Load Average: 0.44, 0.18, 0.16
------------------------------------------------------------------
Benchmark                        Time             CPU   Iterations
------------------------------------------------------------------
singleThread                   269 ms          269 ms            3
directSharing/real_time        276 ms        0.215 ms            3
falseSharing/real_time         268 ms        0.119 ms            3
noSharing/real_time            135 ms        0.152 ms            5
```

发现速度明显上来，约是135ms，是前面几组测试数据的一倍，这是因为修改后的代码，每个变量的内存地址虽然是连续排列，但因为相差64bytes，也就是说不同的变量位于不同的cache line，这样不同的线程都是各自读写不同的cache line，就不会有上述的缓存冲突的问题，整体速度就提升了一倍。

### 写在最后

虽然说缓存解决里了cpu和内存之间的speed gap问题，但因为它的物理结构设计的复杂性，的确会引入一些比较复杂的问题，比如缓存一致性的问题，作为一名程序员，了解一些底层的cpu cache的知识是很有必要的，它会在你解决性能问题的时候提供一些关键性的帮助。

### 参考资料

关于cpu cache的一些资料：

- 维基百科  https://en.wikipedia.org/wiki/CPU_cache
- [Scott Meyers的演讲视频(Cpu Caches and Why You Care)](https://www.youtube.com/watch?v=WDIkqP4JbkE) 
- [What Every Programmer Should Know About Memory](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) 
- https://coolshell.cn/articles/20793.html
- https://manybutfinite.com/post/intel-cpu-caches/
- https://lwn.net/Articles/252125/



性能的benchmark测试方法，讲述了一些关于cpu cache的一些测试思路：

- http://igoro.com/archive/gallery-of-processor-cache-effects/   



利用cache原理知识写出一些高效率的c++代码：

- https://www.youtube.com/watch?v=Nz9SiF0QVKY
- [CppCon 2016: Timur Doumler “Want fast C++? Know your hardware!"](https://www.youtube.com/watch?v=BP6NxVxDQIs&t=3219s)    



MESI Cache Coherency：  

- https://en.wikipedia.org/wiki/MESI_protocol
- https://www.scss.tcd.ie/Jeremy.Jones/vivio/caches/MESIHelp.htm
