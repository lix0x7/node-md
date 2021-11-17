# 引言


这篇文章主要涉及的是 select / poll / epoll ，旨在理清其优缺点，适用场景等。


我们常用的语言是 Java、Node.js，为什么我们需要学习 Linux 的这些系统调用？因为 Java IO 本质上是对操作系统的 IO 的封装。我曾无数次的想去直接学习 Java IO 但是放弃了，认为其概念过于抽象，并不能理解其原理。这样学习，就算最后知道了 API 怎么用，也只是邯郸学步。但其实明白了底层的原理后，再去理解上层的封装就变得简单多了。


这三个**系统调用**是系统层面的 IO 模型，所以放在一起讲。实质上 SELECT 和 POLL 行为基本一致，EPOLL 是一个行为不同的改进版本。下面的英文资料和源码来源于 Linux Programmer's Manual，手册本身会完整的描述函数行为，但此处我只摘出部分我认为对于理解这三个系统调用的关键描述。为了避免翻译过程出出现歧义或信息丢失，此处不翻译任何原文为英文的内容，文章中插入的中文是我自己的一些理解。


# 为什么需要这三个系统调用？


在生产环境的很多应用中，有很多应用都需要同时监控大量的文件描述符，这些文件件描述符可能是文件 IO、socket 等等。但在这个过程中，绝大多数 IO 没有 ready ，如果我们直接去尝试 read 会导致进程挂起。


一个曲线救国的方案是使用多线程去实现监控多个 IO，这样即使挂起了也只是挂起了一个线程，别的线程仍然可以继续工作。


上述方案虽然解决了当前的问题，但在系统中引入了大量的线程，并且线程的切换也是一件极其耗费系统资源的任务。故而，我们希望存在这样的一种系统调用，可以告知我们：对于指定的文件描述符列表，当其中没有 IO ready 的时候，阻塞；一旦有 IO ready 后，返回 ready 的具体 IO，以方便我们编程，并通过将判断 IO 的过程交给内核，提高程序的性能。这就是本文三个系统调用完成的工作。


关于这三个系统调用需要明确和强调的是，他们只是 **IO 多路复用模型**，并没有真正地执行任何 IO 操作，他们只是将 IO 的 ready 状态返回，具体的 IO 操作还是由后续的 read、write 完成的。


# SELECT - Synchronous I/O Multiplexing


`select`函数声明如下，传入参数指明了要读取的文件描述符、要写入的文件描述符、异常、超时间隔。


```c
int select(int nfds, fd_set *readfds, fd_set *writefds,
    		fd_set *exceptfds, struct timeval *timeout);
```


`select()` and `pselect()` allow a program to monitor multiple file descriptors, waiting until one or more of the file descriptors become "ready" for some class of I/O operation (e.g., input possible).  **A file descriptor is considered ready if it is possible to perform a corresponding I/O operation (e.g., read, or a sufficiently small write) without blocking.**


`select()` can monitor only file descriptors numbers that are less than `FD_SETSIZE`; `poll` does not have this limitation.  See BUGS.


`select` 因为使用数组保存文件描述符的，所以最大描述符数量受限于`FD_SETSIZE`这个宏。可以通过重新编译程序的方法修改该值大小，但意义不大。过大的值会导致`select`轮询变慢，影响性能。`poll`通过将存储文件描述符的方式改为链表解决了这个问题，但实际上过多的文件描述符仍会导致性能下降。存储数据结构的变化也是 `select` 与 `poll` 主要区别。


Three independent sets of file descriptors are watched. The file descriptors listed in `readfds` will be watched to see if characters become available for reading (more precisely, to see if a read **will not block**; in particular, a file descriptor is also ready on end-of-file).  The file descriptors in `writefds` will be watched to see if space is available for write (though a large write may still block). The file descriptors in `exceptfds` will be watched for exceptional conditions.  (For examples of some exceptional conditions, see the discussion of POLLPRI in poll.)


The `timeout` argument specifies the interval that `select()` should **block** waiting for a file descriptor to become ready. The call will block until either:


- a file descriptor becomes ready;
- the call is interrupted by a signal handler; or
- the timeout expires.



具体的 API 使用方法、 `fds` 初始化方法、返回值等细节内容参考 The Linux Programming Interface，后文亦同。


从描述中不难看出，`select`仍然会阻塞进程，这不过这个阻塞不是某一个特定的 IO 导致的，而是`select`在等待被监听的一系列文件描述符中的 IO Ready ，如下图：


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100178727-1d489b66-a677-46bc-a60f-e4c59de61dbc.png#align=left&display=inline&height=326&margin=%5Bobject%20Object%5D&originHeight=326&originWidth=609&size=0&status=done&style=shadow&width=609)


## 实现


select 执行过程参考 [Quora - Network Programming: How is select implemented?](https://www.quora.com/Network-Programming-How-is-select-implemented/answer/Kartik-Ayyar?ch=10&share=e2572f2a&srid=zT8xf)


在 select 实现中， `fd_set` 是以位图的方式存储的 IO 类型、文件描述符等信息的，并且 `fd_set` 会扩展到 `[0, nfds)` 内的所有文件描述符。 `nfds` 实际上是给内核的一个用于分配 `fd_set` 大小的提示。在 select 轮询的过程中，内核会遍历 `[0, nfds)` 区间内的所有文件描述符，检验当前文件描述符是否在 `fd_set` 中标记了某一类 IO 事件，如果有，便在内核中设置回调函数并执行后续操作；如果没有，则跳过，检查下一个文件描述符。这个过程可以参考[内核源码 `do_select` ](http://lxr.linux.no/linux+v2.6.36/fs/select.c#L396)。


所以，select 实际上执行的是一个 `[0, nfds)` 区间上的循环操作。也就是说，无论我们想要检查的文件描述符是只有一个 1000 还是 [0, 1000]，select 都会检查从 0 到 1000 的共计 1001 个文件描述符，这也就是 select 性能较差的主要原因。


# POLL - Wait for some event on a file descriptor


`POLL`函数声明如下：


```c
struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    short revents;    /* returned events */
};

int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```


`poll()` 完成的工作和 `select` 十分相似：它等待一系列文件描述符状态转变为 ready，在这期间阻塞。它与 `select` 最大的区别在于入参的形式。另外一个不同在上文已经提到了，就是内部如何存储这一系列的文件描述符。


The set of file descriptors to be monitored is specified in the `fds` argument, which is an array of `pollfd` structures. The caller should specify the number of items in the `fds` array in `nfds`.


If none of the events requested (and no error) has occurred for any of the file descriptors, then `poll()` **blocks** until one of the events occurs.


The `timeout` argument specifies the number of milliseconds that `poll()` should **block** waiting for a file descriptor to become ready. The call will **block** until either:


- a file descriptor becomes ready;
- the call is interrupted by a signal handler; or
- the timeout expires.



与 select 不同的是，poll 通过使用链表存储文件描述符及 IO 事件，这个改变使得 poll 可以监控更多的文件描述符，并且在轮询性能上相比 select 要高一些，尤其是在目标文件描述符稀疏的情况下。


在要监控的文件描述符稀疏的情况下，poll 遍历链表所有节点要比 select 遍历 `[0, nfds)` 上所有的文件描述符性能高得多。以监控 1、10、100、1000、10000 这五个文件描述符为例，poll 只需要关注这五个文件描述，而 select 需要轮询 `[0, 10000]` 上的所有文件描述符。


# EPOLL - I/O event notification facility


The `epoll` API performs a similar task to `poll`: monitoring multiple file descriptors to see if I/O is possible on any of them. The `epoll` API can be used either as an `edge-triggered` or a `level-triggered` interface and **scales well to large numbers of watched file descriptors**.


The central concept of the epoll API is the **epoll instance**, an **in-kernel data structure** which, from a user-space perspective, can be considered as a container for two lists:


- **The interest list** (sometimes also called the epoll set): the set of file descriptors that the process has **registered** an interest in monitoring.
- **The ready list**: the set of file descriptors that are "ready" for I/O.  The ready list is a subset of (or, more precisely, a set of references to) the file descriptors in the interest list that is dynamically populated by the kernel as a result of I/O activity on those file descriptors.



The following system calls are provided to create and manage an epoll instance:


- `epoll_create` creates a new epoll instance and returns a file descriptor referring to that instance.



```c
int epoll_create(int size);
```


- Interest in particular file descriptors is then **registered** via `epoll_ctl`, which adds items to the interest list of the epoll instance.



```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```


- `epoll_wait` waits for I/O events, **blocking** the calling thread if no events are currently available.  (This system call can be thought of as fetching items from the ready list of the epoll instance.)



```c
int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);
```


## Level-triggered and Edge-triggered


# 为什么 epoll 性能好于 poll（select）？


## 初始检查


应用程序每次调用 poll 或 select 时，内核都需要检查一遍当前目标文件描述符的 ready 情况。当循环监控大量的文件描述符时，这部分的性能开销远大于后面两点。但是 epoll 不存在这个检查过程，大大地提升了性能表现。


## 文件描述符的传递


对于 poll 这个系统调用，目标文件描述符列表 `fds` 只存在于每一次调用 poll 的过程中。也就是说，每次调用poll，进程都要重新将 `fds` 拷贝到内核中，内核也需要轮询（也就是 poll 一词的原意）这些文件描述符，然后决定是挂起线程、返回错误或成功返回。所以，对于需要不停的重复执行的 poll 的情况（例如 socket 监听），这样的效率是十分低下的。


对于 epoll 而言，进程通过 epoll_creat 创建了一个新的文件描述符 `epfd` 并将其注册到了内核中。`epfd` 始终存在于内核中，它将进程、目标文件描述符列表 `fds` 的对应关系记录下来，并存储在内核中。随后进程通过 epoll_wait 等待 `epfd` 的状态变化。内核可以在任意 `fds` 中的文件描述符状态发生变化时，直接唤醒相关的进程，避免 O(n) 轮询。在这个过程中，`fds` 是始终存储于内核中的，避免了从用户空间向内核空间拷贝数据的消耗。


## 返回数据的检查


select / poll 返回的数据格式包含了所有的文件描述符，应用程序需要遍历返回结构，取出希望的 IO 事件。相比之下， epoll 的返回结构只包含了 IO 事件，应用程序直接处理这些事件即可。但是，这不是主要的产生性能差距的部分。


# IO多路复用 与 非阻塞IO


通常情况下， **多路复用 IO 模型**都与**非阻塞 IO** 一起使用，这出于一下几个考虑：


- 在 Edge-triggered 的情况下，由于程序只能在发生 IO 事件时获取到一次通知，所以程序应尽可能的处理 IO 数据（例如 read）。在使用阻塞 IO 的情况下，通过循环 read 全部数据后的下一次 read 便会阻塞，此时程序就会被挂起，程序无法重新循环至 `epoll_wait` 等待多个 IO Ready。这种情况如果使用非阻塞 IO ，便可以在没有更多数据可以 read 的情况下跳出 read 循环，重新将等待 IO Ready 的过程交由 epoll 负责。
- 在多线程、多进程程序中，如果某个 IO 事件的数据已经被一个线程读取，此时另外一个线程尝试再次对其进行 IO 便会阻塞。这与上面的情况类似，此时应当使用非阻塞 IO，跳过这次错误的 IO。



# 遗留问题
虽然在大量 IO 的场景下 epoll 性能已经足够优秀，但其本质上仍然是同步 IO，IO 多路复用只是等待 IO 事件的发生，而不处理具体的 IO 读写。IO 读写仍然需要调用方自己调用系统调用将数据从内核态转移到用户态，这个过程无法避免的是阻塞的。解决这种阻塞问题的方案便是异步 IO，当 IO 读写真正完成时，回调调用方的 handler，此时数据已经读写完成，真正避免了调用方阻塞，进一步提高系统可以支持的并发量。Linux 平台在 5.1 发布的 io_uring 技术给 Linux 带来了真正的异步 IO，详情见后续文章。


# Ref


- [Linux IO模式及 select、poll、epoll详解](https://segmentfault.com/a/1190000003063859#articleHeader15)
- [Async IO on Linux: select, poll, and epoll](https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/)
- [Netty序章之BIO NIO AIO演变](https://segmentfault.com/a/1190000012976683)
- [Linux Programmer's Manual  SELECT(2)](http://man7.org/linux/man-pages/man2/select.2.html)
- [Linux Programmer's Manual  POLL(2)](http://man7.org/linux/man-pages/man2/poll.2.html)
- [Linux Programmer's Manual  EPOLL(7)](http://man7.org/linux/man-pages/man7/epoll.7.html)
- [Why is epoll faster than select?](https://stackoverflow.com/a/23198432)
- The Linux Programming Interface - Chapter 63. Alternative I/O Models
