### 问题现象

今天在用gdb调试代码的过程中，遇到一个有意思的问题。

问题是这样的，当我在docker镜像里，使用gdb去attach一个进程，在布置完断点后执行调试程序时，gdb抛出了一个错误：

> "Cannot get thread event message: debugger service failed"

这里说明一下，我使用的gdb的版本为7.6.1，操作系统是centos 7.9。



### 查资料

用谷歌在网上查了一下，找到一篇有价值的资料(完整的资料链接我放在文末)

> I fought with similar gdb issues for a while. My case was having lots of threads spawned that executed few functions and then exited.
>
> It appears if a thread exits too fast and there's lots of these happening sometimes gdb cannot keep up and when it fails, it fails with style as in crashes :) I think it tries to attach to a thread that is already done as per the error message.
>
> I see this as an issue in gdb 6.5 to 7.6 and still happening. Did not try with older versions.
>
> My advice is look for this use case or similar. Once I changed my design to have a thread serving a queue of requests gdb works flawlessly.
>
> Design wise is healthier to have already created threads that digest actions than always spawning new threads.
>
> Still same code debugs without a problem on Visual Studio so I do have to say that is a small disappointment to me with regards to gdb.
>
> I use Eclipse and looking at the GDB traces (usually enabled by default) will give you a better hint of where GDB fails. One of the buttons on the console shows you the GDB trace.

简单解释一下，在程序中创建线程：线程里只是执行几个函数就退出，当在程序中大量的创建这样的线程，导致的一种情况就是gdb的尝试捕获线程信息时出错，表现出来的现象就跟程序崩溃一样，作者在gdb 6.5和7.6的版本上都发现了类似问题。



### 阅读源码

上述是从网上摘的一个资料，我其实也是比较好奇gdb内部到底做了什么，这个错误背后的原因具体是什么？于是我从网上找来了gdb 7.6.1的源码，针对上面的错误信息做了一下搜索，并找到了文件出处：

> ./gdb/linux-thread-db.c:1431:   error (_("Cannot get thread event message: %s"),



这段代码里有几段注释是比较有价值的，我截取了部分代码(完整的函数代码我放到了文章末尾):

```c
/* Check if PID is currently stopped at the location of a thread event
   breakpoint location.  If it is, read the event message and act upon
   the event.  */
static void
check_event (ptid_t ptid)
{
    /* If we have only looked at the first thread before libpthread was
     initialized, we may not know its thread ID yet.  Make sure we do
     before we add another thread to the list.  */
  if (!have_threads (ptid))
    thread_db_find_new_threads_1 (ptid);

  /* If we are at a create breakpoint, we do not know what new lwp
     was created and cannot specifically locate the event message for it.
     We have to call td_ta_event_getmsg() to get
     the latest message.  Since we have no way of correlating whether
     the event message we get back corresponds to our breakpoint, we must
     loop and read all event messages, processing them appropriately.
     This guarantees we will process the correct message before continuing
     from the breakpoint.

     Currently, death events are not enabled.  If they are enabled,
     the death event can use the td_thr_event_getmsg() interface to
     get the message specifically for that lwp and avoid looping
     below.  */

  loop = 1;

  do
    {
      err = info->td_ta_event_getmsg_p (info->thread_agent, &msg);
      if (err != TD_OK)
    {
      if (err == TD_NOMSG)
        return;

      error (_("Cannot get thread event message: %s"),
         thread_db_err_str (err));
    }
}
```

"Cannot get thread event message: debugger service failed"就是check_event函数抛出来的错误，这个函数的作用是读取gdb打断点时的线程信息，但是对于gdb来说，它本身不知道线程什么时候创建，所以在gdb程序里它时通过循环调用td_ta_event_getmsg_p来读取线程信息，再结合上述我们查阅到的资料，不难得出一个猜测：

在一个程序里大量的创建一些短线程(线程内部只是执行几个小函数就结束)，与此同时我们在这些线程的函数体内里埋下一个端点，会不会出现复现上述的问题呢？



### 写测试代码验证

有了上述的猜测，我们可以写一段代码来做一下大胆的验证，测试代码如下：

```c++
#include <thread>
#include <iostream>

int plus(int a, int b) {
    std::cout << a << " + " << b << a + b << std::endl;
    return a + b;
}
int substract(int a, int b) {
    std::cout << a << " - " << b << a - b << std::endl;
    return a - b;
}
int multiply(int a, int b) {
    std::cout << a << " * " << b << a * b << std::endl;
    return a * b;
}
int divide(int a, int b) {
    std::cout << a << " / " << b << a / b << std::endl;
    return a / b;
}
int main(void)
{
    while (true) {
        std::thread t1(plus, 5, 6);
        std::thread t2(substract, 5, 6);
        std::thread t3(multiply, 5, 6);
        std::thread t4(divide, 5, 6);
        t1.join();
        t2.join();
        t3.join();
        t4.join();
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
    return 0;
}
```



运行上述编译的程序，使用gdb attach到进程，执行调试时，果然问题很容易就复现了：

![gdb_bug](http://www.techplorer.cn/upload/2021/04/gdb_bug-6c70da570a4a4374815f778f82d1c4b9.png)



### 总结

虽然上述的问题是来自于gdb的bug，但从中也引申出另外一个问题：那就是去检查一下我们的项目代码里是不是存在大量创建短线程的情况。一般的，我们都尽量避免用这种方式写代码，一方面是用线程池的方案可以解决并发的问题，另一方面反复创建短线程也是一种对资源和性能的消耗。



### 参考资料

- https://stackoverflow.com/questions/16814706/how-does-gdb-attach-to-multi-threaded-process

- ./gdb/linux-thread-db.c:  static void  check_event (ptid_t ptid)

```c
/* Check if PID is currently stopped at the location of a thread event
   breakpoint location.  If it is, read the event message and act upon
   the event.  */

static void
check_event (ptid_t ptid)
{
  struct regcache *regcache = get_thread_regcache (ptid);
  struct gdbarch *gdbarch = get_regcache_arch (regcache);
  td_event_msg_t msg;
  td_thrinfo_t ti;
  td_err_e err;
  CORE_ADDR stop_pc;
  int loop = 0;
  struct thread_db_info *info;

  info = get_thread_db_info (GET_PID (ptid));

  /* Bail out early if we're not at a thread event breakpoint.  */
  stop_pc = regcache_read_pc (regcache)
        - gdbarch_decr_pc_after_break (gdbarch);
  if (stop_pc != info->td_create_bp_addr
      && stop_pc != info->td_death_bp_addr)
    return;

  /* Access an lwp we know is stopped.  */
  info->proc_handle.ptid = ptid;

  /* If we have only looked at the first thread before libpthread was
     initialized, we may not know its thread ID yet.  Make sure we do
     before we add another thread to the list.  */
  if (!have_threads (ptid))
    thread_db_find_new_threads_1 (ptid);

  /* If we are at a create breakpoint, we do not know what new lwp
     was created and cannot specifically locate the event message for it.
     We have to call td_ta_event_getmsg() to get
     the latest message.  Since we have no way of correlating whether
     the event message we get back corresponds to our breakpoint, we must
     loop and read all event messages, processing them appropriately.
     This guarantees we will process the correct message before continuing
     from the breakpoint.

     Currently, death events are not enabled.  If they are enabled,
     the death event can use the td_thr_event_getmsg() interface to
     get the message specifically for that lwp and avoid looping
     below.  */

  loop = 1;

  do
    {
      err = info->td_ta_event_getmsg_p (info->thread_agent, &msg);
      if (err != TD_OK)
    {
      if (err == TD_NOMSG)
        return;

      error (_("Cannot get thread event message: %s"),
         thread_db_err_str (err));
    }

      err = info->td_thr_get_info_p (msg.th_p, &ti);
      if (err != TD_OK)
    error (_("Cannot get thread info: %s"), thread_db_err_str (err));

      ptid = ptid_build (GET_PID (ptid), ti.ti_lid, 0);

      switch (msg.event)
    {
    case TD_CREATE:
      /* Call attach_thread whether or not we already know about a
         thread with this thread ID.  */
      attach_thread (ptid, msg.th_p, &ti);

      break;

    case TD_DEATH:

      if (!in_thread_list (ptid))
        error (_("Spurious thread death event."));

      detach_thread (ptid);

      break;

    default:
      error (_("Spurious thread event."));
    }
    }
  while (loop);
}
```
