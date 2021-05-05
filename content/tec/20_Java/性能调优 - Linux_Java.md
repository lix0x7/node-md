# 影响因素
1. CPU
1. 内存
1. 磁盘IO
1. 网络IO



# 步骤与工具
## CPU
`top` 查看主进程CPU占用， `top -H` 查看线程CPU占用。找到高占用线程TID和名字。


`jps` 获取java进程PID


`jstack <pid> > stack.log` 打印运行栈，搜索高占用线程TID、名字，定位到问题代码。


`vmstat 1` 每隔1s打印一次系统状态，每一列详细解释可参考[https://linux.die.net/man/8/vmstat](https://linux.die.net/man/8/vmstat)：

- procs - r 运行进程数
- procs - b 阻塞进程数
- cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好

![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1608011820214-6fad4b3a-c7b4-49e5-8ceb-7c8ab1fe8e54.png#align=left&display=inline&height=99&margin=%5Bobject%20Object%5D&name=image.png&originHeight=198&originWidth=1492&size=48958&status=done&style=none&width=746)


最后是大杀器：火焰图，分为 `on-cpu` & `off-cpu` ，前者主要关注cpu耗时的代码，后者主要关注io阻塞的代码。下面以 `on-cpu` 火焰图为例。火焰图的解释可参考阮一峰[这篇文章](https://www.google.com/search?q=%E7%81%AB%E7%84%B0%E5%9B%BE&oq=%E7%81%AB%E7%84%B0%E5%9B%BE&aqs=chrome..69i57j0i13i457j0i13l2j69i61l3.880j0j1&sourceid=chrome&ie=UTF-8)。可以使用 [jvm-profiling-tools](https://github.com/jvm-profiling-tools)/[async-profiler](https://github.com/jvm-profiling-tools/async-profiler) 工具快速生成 Java 项目火焰图。


或者：使用使用 [brendangregg](https://github.com/brendangregg)/**[FlameGraph](https://github.com/brendangregg/FlameGraph) **项目生成火焰图，值得一提的是，该项目可以通过jstack生成火焰图，对java项目比较友好。相对而言，使用 `perf` 生成java项目火焰图，操作上比较烦杂。需要注意的是，Java线程的Runnable状态是包含了IO阻塞的，所以对于jstack生成的火焰图，仍然需要注意IO阻塞产生的平顶。下面是一个方便FlameGraph项目使用的脚本：
```shell
export TASK_ID=157248		# 设置压测id
export JAVA_PID=140389	# 设置java项目pid

# 每隔1s抓取一次jstack信息，追加到out.stacks文件
i=0; while (( i++ < 200 )); do jstack ${JAVA_PID} >> out.jstacks; sleep 1; done

# clone项目并且处理jstack文件
git clone https://github.com/brendangregg/FlameGraph.git
cat out.jstacks | ./FlameGraph/stackcollapse-jstack.pl > out.stacks-folded

# 生成火焰图，使用color参数控制色彩高亮，使用width控制生成svg图的宽度
./FlameGraph/flamegraph.pl out.stacks-folded --color java --width 2000 > flame_xinghai_${TASK_ID}.svg

# 把svg发送到本地
sz flame_xinghai_${TASK_ID}.svg

```


## MEM
`free -m` 查看系统可用内存


`jstat -gcutil <pid> 1000` 每隔1000ms打印一次java内存状态，分别显示的是各个内存区域的已用占比、Young GC次数与时间、Full GC次数与时间，总GC时间。如果不需要占比而需要具体值，使用 `-gc` 参数
![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1608011703322-42836a83-8649-46d9-9d71-4ccda103a65f.png#align=left&display=inline&height=105&margin=%5Bobject%20Object%5D&name=image.png&originHeight=210&originWidth=1406&size=50230&status=done&style=none&width=703)


## 磁盘IO
`iostat 1` ：类似 `vmstat 1` ，每隔1s打印一次io状态


## 网络IO
`netstat -apno | grep java` 观察java启动的网络连接状态，主要关注如下几点：

- Recv-Q和Send-Q是否有积压
- State：是否有大量异常的TCP链接
- Timer的长短连接状态是否正常



![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1609323951306-07d240c9-26f1-4e37-9ba1-b78c9f7a6eb9.png#align=left&display=inline&height=432&margin=%5Bobject%20Object%5D&name=image.png&originHeight=864&originWidth=2110&size=325159&status=done&style=none&width=1055)
# 性能瓶颈Checklist

1. 同步日志转化为异步日志
1. 功能链路的涉及到的线程池、链接池大小
   1. 特别注意JDK中的默认实现，是不是用到了ForkJoinPool.CommonPool，这个全局默认线程池默认大小为CPU核数，是为CPU密集场景设计的，不涉及到IO密集场景
   1. 不要使用parallelStream()，这个并行实现使用的是上述ForkJoinPool.CommonPool，全局共享，性能大坑
# Ref

- 阿里Java调优总结 - [https://zhuanlan.zhihu.com/p/92910466](https://zhuanlan.zhihu.com/p/92910466)
- 火焰图生成工具 - [https://github.com/brendangregg/FlameGraph](https://github.com/brendangregg/FlameGraph)
- 一个Java调优记录 - [https://juejin.cn/post/6844903762990104584](https://juejin.cn/post/6844903762990104584)
