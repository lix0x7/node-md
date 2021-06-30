# Why io_uring
两个主要因素：

1. Linux 环境下没有真正的异步 IO，epoll 只是同步非阻塞 IO
1. AIO 的实现和使用问题多多
   - 最大的限制，仅支持 O_DIRECT 标志的无缓冲访问
   - 实际上也只是线程池+阻塞 IO 模拟的异步 IO，并非原生异步 IO
   - API 协议开销过大，对于大量的小规模数据 IO 不友好



详细的原因分析、使用参考官方文档 [Effecient IO with io_uring](https://kernel.dk/io_uring.pdf) 即可，本文不做赘述，原文写的很简练清楚。[【译】高性能异步 IO —— io_uring (Effecient IO with io_uring)](http://icebergu.com/archives/linux-iouring) 是该文的一篇译文，质量很高，可以代替英文原文。


# 与epoll的区别
epoll只负责通知IO完成的事件，不负责具体IO。但是io_uring异步完成了IO操作。


# Reactor / Proactor
这个 [Event Handling Partterns](http://www.dre.vanderbilt.edu/~schmidt/POSA/POSA2/event-patterns.html) 讲的还挺好的，虽然简短了点。


# Ref

- [Lord of the io_uring](https://unixism.net/loti/index.html#) - io_uring 作者的官方网站
- [【译】高性能异步 IO —— io_uring (Effecient IO with io_uring)](http://icebergu.com/archives/linux-iouring)  - 高质量的官方说明文档译文
