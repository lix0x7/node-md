# Java


## Java 基础


### Java的三大特性？


- 封装：把数据和过程封装成一个整体（类），尽可能隐藏细节，只对外提供一系列接口
- 继承：一种代码重用的机制；子类可以定制自己特殊的功能，对于其他的功能可以复用父类已有的实现
- 多态：指相同的函数在不同类型的对象上并获得不同的结果，这种现象称为多态

额外的：
- 抽象：将对象共有的特性抽象为一个独立的部分，限定了一系列接口而不提供具体的实现。在Java中使用abstract关键字修饰，或者使用interface达到抽象的目的

### Java多态及其实现方法？

语法层面实现多态的方法包括：重写、重载和实现接口

虚拟机层面，主要是通过`invokevirtual`这个指令来进行方法调用的。
虚拟机在调用过程中，还会存在静态分派和动态分派的区别，差异在于分派的过程是在编译过程中确定的还是在JVM运行过程中确定的。
其中静态分派主要出现在方法重载时，根据参数类型可以确定待执行的方法；静态分派主要出现在重写场景中，通过继承链查找实际待调用的方法。

ref: [java - Java的多态（深入版）_个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000021936858)

### `==`和hashCode和equals？为什么要重写equals方法？


`==`用于判断两个对象的地址，hashCode用于计算对象的哈希码，而equals方法用于判断两个对象是否是语义上的相等（例如两个对象有不同的地址但是包含了一模一样的成员变量，一般情况下我们都认为这两个对象是equal的）。


因为equals默认由Object对象实现为内存地址，如果我们希望让自己的类实现期望的判断相等的功能，那么需要重写这个方法。


### HashMap中使用可变对象作为key需要注意什么？


要保证可变对象的hashCode是不变的。因为HashMap依赖于hashCode查找，如果hashCode发生了变化，则该键值对就丢失了。


### Java的集合类整体接口和抽象类设计？


最基础接口为Collection，Set/List/Queue接口扩展了Collection接口。


抽象类AbstractCollection实现了Collection接口，AbstractSet/AbstractList/AbstractQueue都继承了AbstractCollection并实现了对应接口。


再往下层就是具体的集合类实现。


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576165041116-22fd2014-2cc7-4a24-871f-0ae4599b4347.png#align=left&display=inline&height=733&margin=%5Bobject%20Object%5D&originHeight=733&originWidth=1077&size=0&status=done&style=none&width=1077)


### ArrayList、LinkedList基础空间、扩展方案？


ArrayList：内部是`Object[]`，初始容量10，每次扩容为会将容量增大至原来的1.5倍，扩容是通过新建一个数组然后拷贝原数据实现的；可以通过trimToSize将容量缩减至数据量大小；也可以通过ensureCapacity扩容至制定容量


LinkedList：双向链表


### HashMap实现？添加过程？rehash过程？Treeify？加载因子？初始容量？resize规则？Hash策略？为什么容量为2的n次幂？put策略？


实现的主体是Node数组：`Node<K,V>[] table;`

JDK1.8及以后考虑到查询效率和存储空间（树节点比链表节点大）两方面原因，链表在到达阈值时会和红黑树相互转化（阈值分别为8和6），这个转化阈值是根据HashCode的泊松分布概率设置的，一个桶中节点达到八个的概率只有小数点后千万分之一。

加载因子默认0.75，初始容量16，每次扩容将桶容量翻倍。

Resize: 1.7以前版本扩容需要重新计算Hash放到新桶里，并且由于是头插法（put和transfer方法都是头插），会导致死锁；1.8及以后版本改成了尾插法（put采用尾插），不需要重新计算Hash，放弃了transfer方法（改为了将一个链表按Hash拆成需要转移到新桶和不需要转移的两部分，不改变Node顺序），避免了之前死循环死锁的情况。下面是1.8中Hashmap的resize过程对于链表的处理：


```java
do {
	next = e.next;
	// 先验条件：oldCap是2的n次幂；
	// 这个判断条件的意思是：如果当前element的hash和oldCap中1的位数不同为1，则说明其`e.hash & (oldCap << 1)`和`e.hash & oldCap`是相等的，也就是说不需要将该节点移动至扩容后的新的桶中。
	// 将一个旧链表通过如上判断拆成了两个新链表，随后存储到新table中对应的位置。
	if ((e.hash & oldCap) == 0) {
		if (loTail == null)
			loHead = e;
		else
			loTail.next = e;
		loTail = e;
	}
	else {
		if (hiTail == null)
			hiHead = e;
		else
			hiTail.next = e;
		hiTail = e;
	}
} while ((e = next) != null);
if (loTail != null) {
	loTail.next = null;
	newTab[j] = loHead;
}
if (hiTail != null) {
	hiTail.next = null;
	newTab[j + oldCap] = hiHead;
}
```


Hash策略：`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)`,这是为了把高位bit传播到低位bit，避免只有低位bit变化时且Bucket较少时的碰撞。


容量设置为2的n次幂有如下几个考虑：


- 取余操作可以通过`hashCode & (capacity-1)`这个位运算来加速，大部分操作也都可以使用位操作加速
- 为了尽可能地减少Hash碰撞
- resize操作依赖于容量翻倍这个先决条件，



### 1.7和1.8的HashMap区别？


- 插入方法：1.7头插，1.8尾插
- resize：1.7会rehash；1.8不会rehash，只判断是否会是否将已有节点放入新桶
- 链表与红黑树相互转化



### HashMap与HashTable区别？


- HashMap是AbstractMap的子类，HashTable是Dictionary子类
- HashMap支持null key和null value
- HashMap非线程安全，Hashtable线程安全（所有方法synchronized修饰）
- HashMap先插入再扩容，HashTable先扩容再插入



### Java中的异常机制？


整体结构图在最后，这里简述一下各个异常的区别：


- Throwable
   - Error：一般是虚拟机自身问题，例如OOM/SOF一类的JVM错误，程序员无能为力
   - Exception
      - RuntimeException
运行时异常，最典型的就是NPE，这类异常一般来说可以通过严谨的变成避免，所以不需要手动检查(check)。
      - IOException/ClassNotFoundException/CloneNotSupportedException
这些异常又总称`Checked Exception`，即出现有可能出现这些异常时一定要显式编程处理(抛出或try/catch处理)，因为这些异常很可能出现，影响程序运行，并且无法通过编程的手段避免。



![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576165041266-62f2f904-ef59-48a1-91a0-75266cbeff53.png#align=left&display=inline&height=731&margin=%5Bobject%20Object%5D&originHeight=957&originWidth=469&size=0&status=done&style=none&width=358)


### BIO/NIO/AIO？todo


### Java8的函数式接口？


简单地说，就是只有一个抽象方法的接口，用于支持lamda表达式，进一步取代大部分场景下的匿名类。JDK中的Runnable/Callable/Comparator都是典型的函数式接口。


Java8提供`@FunctionalInterface`标识函数式接口，但这个注解是非必须的，只要是只含有一个方法的接口都会被JVM判断为函数式接口。这个注解主要用于避免开发人员向接口中添加新的方法。


### Java9的新特性？todo


### Java10新特性？todo


### Java11新特性？todo


## Java多线程


### 线程实现方法？


实现Runnable/Callable接口、继承Thread、线程池


### 什么是线程安全的类？


核心在于正确性。当多个线程访问该类时，无论系统如何调度，线程如何交替，在主代码不进行任何同步操作的情况下，该类总能保证正确的行为


### 如何保证线程安全？（线程同步方法）


- 加锁
- 无锁方法 volatile/CAS
- 使用不变类或线程安全类（Atomic、并发容器）



### AtomicLong？


通过维护一个volatile long值和一组CAS方法（getAndAdd/incrementAndGet）保证原子性的类。


### LongAdder？


类似AtomicLong，但是通过维护一组Cell来应对高并发的情况，一般用并发求和的场景。


### AQS是什么？


抽象队列同步器`AbstractQueuedSynchronizer`主体是一个代表资源的`volatile int state`成员变量和一个存储等待线程的双向链表，以及一组访问对象的方法。


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576165060802-008c8700-23f1-4ef0-9e16-bb36bb2639c2.png#align=left&display=inline&height=401&margin=%5Bobject%20Object%5D&originHeight=401&originWidth=852&size=0&status=done&style=none&width=852)


Doug Lea老爷子设定了三个访问state的方法：


- `getState()`
- `setState()`
- `boolean compareAndSetState(int expect, int update)`



AQS是一个抽象类，其实现了线程等待队列的维护（获取资源失败入队、唤醒出队）等基本操作。


队列的每一个节点存储了线程获取资源的方式（`Exclusive`和`Share`两种）和当前节点的等待状态（不同于线程状态），状态有三种，如下（来源于`AbstractQueuedSynchronizer$Node`注释）。


```java
/**
 * Status field, taking on only the values:
 *   SIGNAL:     The successor of this node is (or will soon be)
 *               blocked (via park), so the current node must
 *               unpark its successor when it releases or
 *               cancels. To avoid races, acquire methods must
 *               first indicate they need a signal,
 *               then retry the atomic acquire, and then,
 *               on failure, block.
 *   CANCELLED:  This node is cancelled due to timeout or interrupt.
 *               Nodes never leave this state. In particular,
 *               a thread with cancelled node never again blocks.
 *   CONDITION:  This node is currently on a condition queue.
 *               It will not be used as a sync queue node
 *               until transferred, at which time the status
 *               will be set to 0. (Use of this value here has
 *               nothing to do with the other uses of the
 *               field, but simplifies mechanics.)
 *   PROPAGATE:  A releaseShared should be propagated to other
 *               nodes. This is set (for head node only) in
 *               doReleaseShared to ensure propagation
 *               continues, even if other operations have
 *               since intervened.
 *   0:          None of the above
 *
 * The values are arranged numerically to simplify use.
 * Non-negative values mean that a node doesn't need to
 * signal. So, most code doesn't need to check for particular
 * values, just for sign.
 *
 * The field is initialized to 0 for normal sync nodes, and
 * CONDITION for condition nodes.  It is modified using CAS
 * (or when possible, unconditional volatile writes).
 */
volatile int waitStatus;
```


其实现类主要需要实现state的获取与释放方式（公平/非公平、读写锁等），需要重写的方法如下：


- `isHeldExclusively()`：判断当前线程是否正独占资源
- `tryAcquire(int)`：独占获取
- `tryRelease(int)`：独占释放
- `tryAcquireShared(int)`：共享获取
- `tryReleaseShared(int)`：共享释放



AQS已经实现了一些顶层基础方法：


- `acquire(int)`
负责获取资源、入队操作。

```java
public final void acquire(int arg) {
	if (!tryAcquire(arg) &&
		acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
		// selfInterrupt就是Thread.currentThread().interrupt();
		selfInterrupt();
}

private Node addWaiter(Node mode) {
	// 以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
	Node node = new Node(Thread.currentThread(), mode);
	
	// 尝试快速方式直接放到队尾。
	Node pred = tail;
	if (pred != null) {
		node.prev = pred;
		if (compareAndSetTail(pred, node)) {
			pred.next = node;
			return node;
		}
	}
	
	// 上一步失败则通过enq入队，enq就是一个自旋操作
	enq(node);
	return node;
}

// 在队列中时自旋检查当前线程是被唤醒的，并尝试获取资源，这个函数名其实叫做acquireInQueue更直观一些
final boolean acquireQueued(final Node node, int arg) {
	boolean failed = true;// 标记是否成功拿到资源
	try {
		boolean interrupted = false;// 标记等待过程中是否被中断过
		
		// 自旋
		for (;;) {
			final Node p = node.predecessor();	// 拿到前驱
			// 如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
			if (p == head && tryAcquire(arg)) {
				setHead(node);	// 拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
				p.next = null; 	// setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
				failed = false;
				return interrupted;	// 返回等待过程中是否被中断过
			}
			
			// 如果自己可以休息了，就进入waiting状态，直到被unpark()
			if (shouldParkAfterFailedAcquire(p, node) &&
				parkAndCheckInterrupt())
				interrupted = true;	// 如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576165098855-5dceae9c-2f37-49f3-8f50-59da145db96e.png#align=left&display=inline&height=232&margin=%5Bobject%20Object%5D&originHeight=232&originWidth=1076&size=0&status=done&style=none&width=1076)

- `release(int)`
负责释放资源、唤醒队列中的后继线程。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
	// 这里，node一般为当前线程所在的结点。
	int ws = node.waitStatus;
	if (ws < 0)// 置零当前线程所在的结点状态，允许失败。
		compareAndSetWaitStatus(node, ws, 0);

	Node s = node.next;// 找到下一个需要唤醒的结点s
	if (s == null || s.waitStatus > 0) {// 如果为空或已取消
		s = null;
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)// 从这里可以看出，<=0的结点，都是还有效的结点。
				s = t;
	}
	if (s != null)
		LockSupport.unpark(s.thread);// 唤醒
}
```




参考[Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)


### ReentrantLock实现？


委托给AQS实现，通过重写了AQS的锁获取和释放策略提供了不同形式的锁。


详情参考源码和这篇文章：[从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)。


### synchronized作用对象？

类静态方法（类锁）、类（类锁）、实例方法（对象锁）、实例（对象锁）

### synchronized实现原理

通过JVM中的monitor实现的，相关JVM字节码操作为`monitorenter` / `monitorexit`。

在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++实现的，由ObjectMonitor实现的。每个对象中都内置了一个 ObjectMonitor对象。

另外，wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

ref: [Java 并发常见面试题总结（中） | JavaGuide](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#%E8%AE%B2%E4%B8%80%E4%B8%8B-synchronized-%E5%85%B3%E9%94%AE%E5%AD%97%E7%9A%84%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86)


### ReentrantLock和synchronized异同？


最大的区别是一个是锁，一个是Java的关键字。


实现方式上ReentrantLock使用的是`LockSupport`提供的`park`/`unpark`方法实现阻塞和释放阻塞进程；synchronized使用的是监视器`Monitor`实现的阻塞与释放


synchronized可以自动获取释放，ReentrantLock需要手动释放，如果忘记释放或异常时未释放可能会死锁。


除此以外，ReentrantLock可以有很多参数：中断取锁、多条件、公平锁


### 常见的锁优化方法？


- 自旋锁
- 锁粗化：一个大锁优于多个小锁
- 偏向锁：消除在无竞争环境下的同步原语，判断依据来自于逃逸分析
- 锁消除：JVM编译优化，将不存在竞争的锁消除


### 轻量级锁和重量级锁是什么？锁膨胀过程是怎样的？


简而言之，重量级锁是使用操作系统互斥量实现的锁；轻量级锁是指在低冲突情况下，通过CAS方法避免互斥锁一类的重量级锁而造成的大量开销，其加锁、检查过程如下：


每一个对象头会有一个`Mark Word`的区域，其中有2bit记录了锁定状态，默认为未锁定`01`。`Mark Word`是一个非固定数据区域，在不同情况下储存了不同信息。


1. 代码进入同步块时，在当前线程的栈帧中创建一个对象的`Mark Word`的副本，称为`Lock Record`
2. 然后通过CAS将`Mark Word`修改为栈帧上的`Lock Record`的引用，如果成功进行步骤3，如果失败进行步骤4
3. CAS写入成功，表明当前线程拥有了该对象的锁，此时将对象的锁定状态修改为轻量级锁`00`
4. CAS写入失败，JVM先检查当前线程是否已经获得了轻量级锁，如果有则继续执行；如果没有，则表明发生了冲突，此时进行锁膨胀将轻量级锁升级为重量级锁，即将锁定状态置为重量级锁`10`

![](.Java%20面试相关.assets/2022-09-13-22-50-48.png)

### 偏向锁实现？


在轻量级锁的基础上还可以进一步降低开销，它将第一个获取对象锁的线程的ID通过CAS方法写入`Mark Word`，随后JVM每次在该线程下使用该类可以不再使用CAS检查。一旦有其他线程介入，则偏向模式结束，进入轻量级锁状态。


### 为什么wait、notify、notifyAll方法定义在Object类中？


因为所有对象都要持有锁，每一个类对象实质上是一个`Monitor`，这些方法都是`Monitor`的实现。


### 为什么wait、notify、notifyAll方法必须在同步块中调用？


因为这三个方法依赖于进入同步块时构建的`Monitor`来维持正确的行为。


### 线程池原理？常见的线程池？


线程池维护了一组工作线程`HashSet<Worker> workers`、一个用于存放任务的阻塞队列`BlockingQueue<Runnable> workQueue`。线程池可以避免频繁的线程创建与销毁，降低系统资源的消耗；固定的线程数量也方便管理。


JDK推荐使用`Executors`类的静态工厂方法创建常见的线程池，这些线程池实际上都是`ThreadPoolExecutor`的参数不同的实例，常见的线程池如下：


- FixedThreadPool：创建固定线程数的线程池
- SingleThreadExecutor：创建只有一个工作线程的线程池
- CachedThreadPool：线程上下限分别为0和Integer.MAX_VALUE，也就是说线程是无限的；没有阻塞队列，每一个新任务必须等到有空闲线程，否则阻塞。
- ForkAndJoinPool
- 线程池调优：
   - 设置最大线程数，防止系统资源耗尽
   - 针对不同类别任务设置不同线程数：CPU型任务（线程数量=内核数量），IO型任务（线程数量=内核数量*2）



### 线程池的参数都有什么？


直接贴线程池类`ThreadPoolExecutor`的注释了。


```java
/**
* Creates a new {@code ThreadPoolExecutor} with the given initial
* parameters.
*
* @param corePoolSize the number of threads to keep in the pool, even
*        if they are idle, unless {@code allowCoreThreadTimeOut} is set
* @param maximumPoolSize the maximum number of threads to allow in the
*        pool
* @param keepAliveTime when the number of threads is greater than
*        the core, this is the maximum time that excess idle threads
*        will wait for new tasks before terminating.
* @param unit the time unit for the {@code keepAliveTime} argument
* @param workQueue the queue to use for holding tasks before they are
*        executed.  This queue will hold only the {@code Runnable}
*        tasks submitted by the {@code execute} method.
* @param threadFactory the factory to use when the executor
*        creates a new thread 用于设定线程优先级、名字、分组、是否为守护线程等
* @param handler the handler to use when execution is blocked
*        because the thread bounds and queue capacities are reached
* @throws IllegalArgumentException if one of the following holds:<br>
*         {@code corePoolSize < 0}<br>
*         {@code keepAliveTime < 0}<br>
*         {@code maximumPoolSize <= 0}<br>
*         {@code maximumPoolSize < corePoolSize}
* @throws NullPointerException if {@code workQueue}
*         or {@code threadFactory} or {@code handler} is null
*/

public ThreadPoolExecutor(int corePoolSize,
                        	int maximumPoolSize,
                        	long keepAliveTime,
                        	TimeUnit unit,
                        	BlockingQueue<Runnable> workQueue,
                        	ThreadFactory threadFactory,
                        	RejectedExecutionHandler handler);
```


### 线程池提交任务过程？


援引JDK1.8`ThreadPoolExecutor`注释：


> When a new task is submitted in method {[@link ]() #execute(Runnable)}, and fewer than corePoolSize threads are running, a new thread is created to handle the request, even if other worker threads are idle.  If there are more than corePoolSize but less than maximumPoolSize threads running, a new thread will be created only if the queue is full.  By setting corePoolSize and maximumPoolSize the same, you create a fixed-size thread pool. By setting maximumPoolSize to an essentially unbounded value such as {[@code ]() Integer.MAX_VALUE}, you allow the pool to accommodate an arbitrary number of concurrent tasks. Most typically, core and maximum pool sizes are set only upon construction, but they may also be changed dynamically using {[@link ]() #setCorePoolSize} and {[@link ]() #setMaximumPoolSize}.



### 线程池的reject策略？


四种：


- `AbortPolicy`：抛出`RejectedExecutionException`异常
- `CallerRunsPolicy`：使用调用者线程执行
- `DiscardPolicy`：丢弃这个提交的任务
- `DiscardOldestPolicy`：丢弃队列中最老的任务，然后重新提交任务（可能重复触发该策略）



### 线程池运行状态？


- `RUNNING`：接收新任务并处理队列积压的任务
- `SHUTDOWN`：不再接收新任务，但是会继续处理队列中的任务。调用`shutdown()`进入该状态
- `STOP`：中断正在执行的任务，不再接收新任务，不再处理队列中的任务。调用`shutdownNow()`进入该状态
- `TIDYING`：所有的任务都结束后进入`TIDYING`状态，执行`terminate()`方法
- `TERMINATED`：`terminated()`执行结束



### HashMap多线程下为什么死锁？todo，动图


JDK1.7使用头插法，JDK1.8采用尾插法。采用头插法并且有并发时如果发生了扩容，可能会导致桶上的链表出现环。采用尾插法后不会出现这个问题了，因为只有一次遍历过程。


### ConcurrentHashMap实现？1.7和1.8实现区别？


- JDK1.7及其之前
Segment+Bin+Entry链表，进行插入和写入操作
   - Entry链表的读写机制类似于CopyOnWrite容器。数据类型的key、next声明为volatile变量，put操作是在每一个链表的链头插入；remove操作不修改源链表的而仅修改指针
   - 写操作时会对当前桶所对应的链表加锁（实际实现中是对链表的第一个节点加锁，为了不浪费空间去维护一个锁），但是这个锁不影响读操作，只影响其他写操作。特别的是，当一个桶的链表不存在时，该类是通过CAS的方式插入第一个节点的。
   - 弱一致性


详情见[Here](https://blog.csdn.net/justloveyou_/article/details/78653929)的第31项

- JDK1.8及其以后
基本结构和Map相同，但是table本身（Node数组）和table内的Node的val、next都是volatile修饰的；除此以外，设置table对象是通过compareAndSwap方法实现的，这保证了线程安全性。
put过程中会如果对应桶的头节点不存在，则通过CAS方法写入如节点；否则锁住该头节点，然后插入，这个锁也只对其他写操作有效，而不锁定读操作。
resize过程（ConcurrentHashMap中称为Transfer）：resize思想同HashMap，但是会加锁并将节点状态标记为resize中，此时如果有其他线程访问到该节点，则会帮助resize。
获取size过程：维护了一个counterCells数组，用和LongAdder相同的逻辑（Cells），降低并发冲突。



### HashMap和ConcurrentHashMap有什么不同？


- 线程不安全/线程安全
- 获取size的方法不同
- 扩容方法不同



### CopyOnWrite容器是什么？如何实现的？什么叫弱一致性？为什么读不需要加锁？适用场景？


### 快速失败`fail-fast`和安全失败`fail-safe`？


### ThreadLocal是什么？怎么实现的？为什么会导致内存泄漏？


多个线程访问同一个对象时，如果希望每个线程使用对象成员的一个独占副本，可以将通过ThreadLocal达成这个目的。


其实ThreadLocal只是一个管理类，真正具体的内容存放的位置是线程实例的`threadLocals`对象里，也就是`thread.threadLocals`，其类型为`ThreadLocal.ThreadLocalMap`。所以说，数据是存在线程对象里的，而非ThreadLocal对象。`ThreadLocalMap`的`Entry`的key是ThreadLocal对象，value是对应线程存储的副本。也就是说，ThreadLocal成员变量仅仅是一个查询线程变量副本的一个标志，此处引用是弱引用，保证ThreadLocal机制不会影响到这个标志的回收。


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576165116890-7c1688bc-75a1-4952-9322-09cf95afb403.png#align=left&display=inline&height=403&margin=%5Bobject%20Object%5D&originHeight=403&originWidth=714&size=0&status=done&style=none&width=714)


但是这样会产生内存泄漏的问题，如果key指向的ThreadLocal对象已经回收，但是线程一直存在，这就导致了`thread->map->entry(value)->Object`的对象存在一串强引用关系而无法回收这个已经无法访问的对象，造成内存泄漏。比较好的解决方法是在使用完后手动移除这个value指向的对象。


### volatile 的两个特性？底层实现（具体到x86汇编）？

volatile解决的是线程之间可见性的问题，有两个特性：

- 立即可见
- 禁止指令重排序

在 JVM 层面，禁止指令重排序就是一项 happens-before 原则的具体体现，通过内存屏障实现。happens-before 规定：对 volatile 变量的写操作一定在读操作之前。
默认的happens-before只规定了单线程之内普通变量的顺序，但没有规定要求多线程下普通变量需要满足写后读的要求，但是volatile是严格要求了的。

在 x86 层面，JIT 会在编译 Java 字节码时，如果检测到了变量有 volatile 修饰，则会在写操作前添加 lock 指令（体系结构级别的内存屏障），这个指令有如下两个作用（其实 JVM 做的事情和 CPU 是相同的）：

- 将当前缓存的值立刻写回主内存
- 使所有 CPU 里其他涉及到该内存地址的缓存无效


参考[聊聊并发（一）——深入分析 Volatile 的实现原理](https://infoq.cn/article/ftf-java-volatile)


### 如何中断线程？interrupt做了什么？


### 并发编程三大特性？

- 原子性
- 可见性：volatile
- 有序性

## JVM

### JVM内存模型（JMM），什么是JMM，为什么需要JMM？

JMM是一个定义了Java内存行为的规范，目标是去除Java运行在不同操作系统、硬件时产生的行为差异。

简单来讲，整体JVM有一个共用的主内存，每一个线程有自己的工作内存（对应CPU Cache）。内存在读写数据前必须将主内存中的数据读取到工作内存才能进行下一步操作。

这期间有八个基本操作分别用来读取、写入、加解锁变量：

- 读取：read/load/use
- 写入：assign/store/write
- 加锁：lock/unlock

除此以外，还定义了happens-before行为



### JVM内存结构？

JDK 8之后的版本有五个主要部分：程序计数器、虚拟机栈、本地方法栈、元空间、堆区；堆区在具体实现的时候又会分为老年代、新生代，以实现不同的垃圾回收算法。

![](.Java%20面试相关.assets/2022-09-13-23-07-45.png)


### GC如何确定要被回收的对象？

通过引用计数法（难以解决循环引用的问题，但是快）和可达性分析（通过分析`GC Roots`的引用链确定需要回收的对象，缺点是运行效率差一些，但是准确），JVM实现一般使用可达性分析的方法。


### 有哪些 `GC Roots` ？


作为GC Roots的对象的生命周期都是与他们所属线程（栈帧中的对象）、类的生命周期（类的静态成员）相同的：


- 虚拟机栈帧中本地变量表引用的对象
- 本地方法栈中Native方法引用的对象
- 元空间中类静态属性引用的对象
- 元空间中常量引用的对象



### GC是如何回收空间（三种）？


- 标记清除：会留下外部碎片
- 复制算法：用于新生代。过程是将内存分成三部分，每次都是将存活对象从其中两块（一块Eden和一块Survivor）复制到另外一块（Survivor）。
- 标记整理：就是标记清除，然后把内存规整，消去碎片。好处是不会产生碎片，缺点是停顿时间更长



### 何时会触发GC？


- `Young GC`：小对象分配在新生代但内存不足时，触发`Young GC`
- `Full GC`：`Minor GC`前，VM需要先检查老年代剩余空间大小是否足够容纳新生代所有对象，这是为了避免极端情况下所有对象都不能被回收且另一个Survivor区不足以容纳所有对象，而需要经过空间分配担保使新生代对象直接进入老年代的情况，如果不足则触发`Full GC`



### 有哪些垃圾回收器？


最基本的是Serial分别工作在新生代和老年代的两个版本，`Serial`/`Serial Old`都会完全`stop the world`然后单线程进行GC再恢复用户线程


随后的改进版本主要是将GC过程并行化的Parallel，也有分别工作在新生代和老年代的两个版本。`ParNew`/`Parallel Old`也会完全`stop the world`，他们和Serial GC区别就是将GC过程变成多线程的。虽然叫Parallel，但并不是只得和用户线程并行，而只是自身的多个GC线程并行。


除了上述四种外，还有CMS和G1收集器，CMS工作在老年代，需要配合其他的新生代垃圾回收器；G1同时工作在新生代和老年代。


### 介绍下CMS？


CMS是一个工作在**老年代**的垃圾收集器，分为如下四个阶段进行垃圾回收：


- 初始标记，`STW`
标记`GC Roots`及其直接引用的对象
- 并发标记，并发
与用户线程并行，遍历引用树
- 重新标记，`STW`
修正并发标记过程中出现引用关系变动的对象的标记
- 并发清理，并发
使用标记清除的方法进行垃圾回收。



优点：


- 并行的GC可以减少应用停顿



缺点：


- 对CPU资源敏感，如果本身CPU资源不充裕，反而会增加停顿
- 使用标记清除算法，会产生外部碎片，提高分配内存消耗。如果清理后仍然没有足够空间分配对象，则会临时调用`Serial Old`进行标记整理，产生更大的GC延迟



### 介绍下G1？比CMS强在哪？


G1取意`Garbage First`，G1在传统GC上做了很大的改进，主要是为了降低停顿时间，这些主要改动如下：


- 同时工作在新生代和老年代，但是他们在物理上不再是隔离的了
- 将堆区切分为块`Region`，整体基于标记整理算法，块之间采取复制算法
- 维护了一个由垃圾占比组成的`Region`列表，可以根据设定的最大`STW`时间优先回收垃圾占比的多的`Region`
- 每个`Region`维护`Rembered Set`保存引用自己的对象所在的`Region`，并将Region内的对象直接加入`GC Roots`减少扫描时间



相比CMS的改进主要有如下几点:


- 同时支持新生代和老年代的垃圾回收
- 可以设置最大停顿时间，在固定的时间内回收尽可能多的对象
- 基于 compact 算法可以避免产生外部碎片



回收过程和CMS类似，分别为`初始标记（阻塞）->并发标记（并发）->重新标记（阻塞）->垃圾回收（阻塞）`


### 为什么区分新生代老年代？


新生代对象的生命周期都很短，需要频繁GC，但老年代不需要频繁GC。对不同区域的对象使用不同的收集器可以提升JVM执行效率。


### 为什么Eden和Survivor的比例是8:1？为什么有两块Survivor？


很多实验表明新生代的每次GC只会有不到10%大小的对象存活，所以留出这部分空间即可，这是一个经验值。


两块Survivor可以只复制小部分数据，一是可以通过直接修改指针的方法避免清除数据的操作消耗；二是可以通过复制的方法避免内存中产生外部碎片。


### 什么时候对象会进入老年代？


- 大对象直接分配，可以参数设定
- 分配担保
- 年龄达到15的阈值
- Survivor中的相同年龄对象占Survivor一半空间以上



### 类加载流程？


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1576165131295-09401bc3-a3ca-4c6c-bc1f-425f00919f77.png#align=left&display=inline&height=223&margin=%5Bobject%20Object%5D&originHeight=223&originWidth=627&size=0&status=done&style=none&width=627)


1. 加载：通过类全全限定名加载二进制字节流，不仅限于文件
2. 验证：验证版本、安全性
3. 准备：JVM分配方法去的内存空间并初始化类变量初始值，此处只对类变量（static）进行操作。需要注意的是这里的初始值只是0值，真正在代码中设置的值需要在初始化步骤中执行`<clinit>`
4. 解析：将类文件中的引用从全限定名解析成真正的在JVM中对应的引用
5. 初始化：执行类初始化方法`<clinit>`，`<clinit>`方法是编译器收集类中的类变量赋值和静态语句块中的语句合并生成的。JVM保证这个方法的调用时线程安全的



### 双亲委派模型？为什么需要双亲委派模型？怎么破坏双亲委派模型？为什么需要破坏？


JVM中的类加载器层级如下：启动类加载器(`<JAVA_HOME>/lib`)<-扩展类加载器(`<JAVA_HOME>/lib/ext`)<-应用程序类加载器(默认类加载器，`<CLASSPATH>`)<-自定义类加载器。


双亲委派模型指的是除了启动类加载器，其他所有类加载器都会先将自己的加载请求委托给符加载器，如果父加载器加载失败，才会尝试自己加载。


为什么需要？双亲委派模型的意义在于避免自行实现的类覆盖了核心类库而导致功能异常，或是防止被恶意修改的类影响JVM运行安全。


怎么破坏？通过自行实现ClassLoader直接加载类而不先委托给父加载器。


为什么需要破坏？

## 其他技术

### AOP 原理？
// todo

1. 编译期字节码写入
2. 对于非 `final class` 在运行时继承类实现
3. 对于 `final class` 使用 jdk 动态代理实现


# 算法与数据结构


## 快排、堆排、归并、希尔、桶排序，实现？复杂度？最差情况？优势、适用场景？


除了桶排序以外都为比较算法。

| 算法 | 稳定性 | 时间复杂度 | 最好情况时间复杂度 | 最坏情况时间复杂度 | 空间复杂度 |
| --- | --- | --- | --- | --- | --- |
| 冒泡排序 | 稳定 | $$O(n^2)$$ | - | - | $$O(1)$$ |
| 快速排序 | 不稳定 | $$O(n \lg n)$$ | $$O(n)$$ | $$O(n^2)$$ | $$O(1)$$ |
| 堆排序 | 不稳定 | $$O(n \lg n)$$ | $$O(n)$$ | $$O(n^2)$$ | $$O(n)$$ |
| 归并排序 | 稳定 | $$O(n \lg n)$$ | - | - | $$O(n)$$ |
| 桶排序 | 稳定 | $$O(n)$$ | - | - | $$O(n)$$ |



## 二叉树->二叉搜索树->平衡二叉搜索树->红黑树->B树->B+树，这个递进过程中的所有解决的问题和改进方法？


二叉树中数值是无序的，查找需要遍历整棵树。

二叉搜索树保证了$$O(\lg n)$$时间复杂度的查找。

但是二叉搜索树在极端情况下会转化成链表，查找速度降为$$O(n^2)$$，平衡二叉搜索树通过旋转的操作保证左右子树高度差不超过1，也就保证了查找时间复杂度始终控制在$$O(n \lg n)$$。

平衡二叉搜索树虽然保证了平衡，但是维持平衡的旋转操作代价有可能很高。红黑树是平衡二叉搜索树的简化实现，它并不严格要求左右子树高度差不超过1。这将旋转操作降低至最多三次，提高了效率。红黑树的统计性能强于平衡二叉搜索树，它通过**变色**和**旋转**两个操作维持如下的这些性质：

- 每个节点要么是红色，要么是黑色；
- 根节点永远是黑色的；
- 所有的叶节点都是空节点（即 null），并且是黑色的；
- 每个红色节点的两个子节点都是黑色（从每个叶子到根的路径上不会有两个连续的红色节点）；
- **从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点**。该性质保证了最长路径不会超过任何其他路径的两倍

B树是应对红黑树这类二叉树在大数据量的情况下深度太大的问题而创造的**多路查找树**。通过允许每个节点下可能有更多的子节点以减小树深度，降低查找的统计时间。

B+树是优化了的B树，他的所有数据都存放于叶节点，这意味着连续的数据都是存放在相近的位置的，利于磁盘这类存储器的顺序访问。除此以外，B+树还将叶节点串联起来，这样扫描全表的操作就等同于链表遍历，而B树需要搜索整棵树。


## 90G文件，10G内存，怎么排序？


外排序。


1. 先将90G文件分为10G的块，每块加载到内存中快排后写回。
2. 将内存分为10块，取每个有序块中最小的1G加载到内存，剩余1G做归并缓冲区
3. 在内存中进行归并排序，写入归并缓冲区，如果归并缓冲区满了则写出到磁盘，随后清空归并缓冲区；如果某个数据块都读完了，则按序到磁盘中读取新的数据块到内存中，重复上述过程直到所有磁盘中的数据都已排序完毕


# 操作系统


## 进程、线程


### 进程和线程是什么？


- 进程时程序的运行时，是系统分配资源的基本单位
- 线程是进程内的子任务，是系统分配时间片的基本单位



### 进程、线程区别？


- 线程依赖于进程存在而存在
- 进程是资源分配的最小单位；线程是时间片调度的最小单位
- 进程独享内存空间；线程没有自己的内存空间，而是共享进程的内存空间



### 进程间通信方式（IPC，Inter-Process Communication）


- 文件
- 管道
   - 匿名管道：FIFO，半双工；生命周期仅限于创建该管道的进程，多用于父子进程间通信
   - 命名管道：FIFO，利用文件系统实现；生命周期可与操作系统一样；可用于无亲缘关系的进程通信
- 信号
一种异步的通知机制。当一个信号发送给一个进程，操作系统中断了进程正常的控制流程，此时，任何非原子操作都将被中断。如果进程定义了信号的处理函数，那么它将被执行，否则就执行默认的处理函数。
- 消息队列
- 共享内存
- 套接字
- 信号量
其实信号量并不是进程通信手段，而是线程间用于实现锁的同步控制方法



参考自[WIKI_IPC](https://zh.wikipedia.org/wiki/%E8%A1%8C%E7%A8%8B%E9%96%93%E9%80%9A%E8%A8%8A)


### 进程调度策略？


- FCFS 先来先服务，非抢占
- SJF 最短作业优先
- 时间片轮转 可抢占也可不抢占
- 优先级调度（抢占、非抢占）
- 优先级、时间片、抢占式的组合



### 进程间同步方式？


- 信号量和PV操作
通过信号量和PV操作来协调线程的同步。信号量取值范围只有0和1的时候就是互斥锁。
- 管程（或称作为监视器，Monitor）
管程是为了解决大量信号量和PV操作散布不易管理的问题而提出的。
管程包含了临界资源和对这些资源操作的一组过程。这些临界资源只能通过管程中的过程访问而不能直接访问。且同一时间管程中只会有一个进程在执行。
Java已经通过Object对象上的wait/notify/notifyAll三个方法实现了管程。



## 死锁


### 什么是死锁？


在两个或更多线程并发中，每个线程都等待着其他线程持有的资源并且不能放弃自己已经持有的资源，导致系统状态不能继续向前推荐，这种状态称为死锁。


### 死锁产生的条件？


- 资源互斥
- 请求与保持
- 循环等待
- 不剥夺



### 怎么处理死锁？


通常从四个角度处理死锁:


- 死锁预防
使系统在构建之处就破坏死锁产生的条件进行预防。
- 死锁避免
在资源分配过程中，防止系统进入不安全状态。所谓不安全状态指的是找不到一个分配资源的序列使得所有每个线程都能顺利完成。
使用**银行家算法**。
- 死锁检测
运行时出现死锁，能及时发现死锁。
使用**资源分配图化简**。
- 死锁解除
发生死锁后，操作系统可以撤销进程，回收资源，使系统解除死锁状态。



### 银行家算法是什么？


银行家算法指的是系统会持有所有的资源，每个进程都要向操作系统告知自己**需要的资源的最大量**、**每次申请需要的申请量**，并且保证在**有限时间内归还资源**。操作系统会在进程申请资源时检查剩余的资源和进程已经持有的资源是不是满足了进程需要资源的最大值，如果满足，证明可以分配给该进程资源且进程可以安全执行完成并返回这些资源；如果不满足则拒绝分配资源。这个算法使系统避免了进入不安全状态。


### 资源分配图是什么？


资源分配图代表着进程对资源的请求情况，如果资源分配图化简后有边剩余，则代表系统中出现了死锁。用于死锁检测。参考[操作系统原理_北京大学_Coursera](https://www.coursera.org/lecture/os-pku/zi-yuan-fen-pei-tu-JvKcU)。


InnoDB就是通过类似的等待图实现的死锁检测，如果发现了死锁就会把涉及到的事务中权重最低的一个回滚。


## 虚存、页式存储


### 虚拟内存的寻址过程？


（虚拟地址）->TLB->页表->（物理地址）->Cache->内存


### 什么是缺页中断？页面置换算法？抖动是什么？


缺页中断指的是应用访问的虚存对应的实页并不在内存中的情况，此时需要页面置将该虚页置换到内存中。


页面置换算法有FIFO、LRU、OPT。


抖动指的是虚页频繁换进换出，一般出现在实际内存空间不足的情况。


## Linux


### 常用指令，参考Linux章节


# 设计模式


## 设计模式的七大设计原则？


## 单例模式？Java的几种写法？


- 直接加载

```java
public class Singleton{
	private static Object singleton = new Object();
	public static synchronized Object getSingleton(){
		return singleton;
	} 
}
```

- 优点：写法简单粗暴
- 缺点：每次读都要加锁，浪费时间；并且直接创建了对象，浪费空间

- 懒加载
```java
public class Singleton{
	private static Object singleton = null;
	public static synchronized Object getSingleton(){
		if (singleton == null){
			singleton = new Object();
		}
		return singleton;
	}
}
```

- 优点：实现简单；懒加载，节约资源
- 缺点：每次获取单例对象都要加锁，效率低

- 双重检查

```java
public class Singleton{
	private static volatile Object singleton = null;
	public static Object getObject(){
		if (singleton == null){
			synchronized (Singleton.class){
				if (singleton == null){
					singleton = new Object();
				}
			}
		}
		return singleton;
	} 
}
```

优点：懒加载；只在初始化的时候才加锁
缺点：每次都要进行先判断

- 静态内部类

```java
public class Singleton{
	//private构造方法，避免构造多个单例对象
	private Singleton(){}

	public static Object getSingleton(){
		return Holder.singleton;
	}

	class Holder{
		public static final Object singleton = new Object();
	}		
}
```

优点：实现简单、依赖于JVM实现懒加载效率高

- enum

```java
public enum Singleton{
	HOLDER();

	public Object singleton = new singleton();
	Singleton(){}
	public Object getSingleton(){
		return this.singleton;
	}
}
```


## 为什么上面的双重检查的方法要使用volatile关键字？

JVM创建对象分为三个阶段：

1.栈上分配引用空间、堆上分配内存空间
2.初始化对象
3.将引用指向堆上对象

后两步可能会被重排序，那么假若此时有线程B对调用Singleton.getInstance方法，那么在第一次检查时会认为引用已经存在并返回了这个引用，但是此时返回的引用指向的对象还未完全初始化，例如某些字段值还未初始化完毕。这样的对象处于不确定的状态，此时如果调用这个不确定状态的对象则会抛出异常。

但双重检查已经是一种过时的方法（Java 并发编程实践在 2006 年、甚至更早就提到了），主要因为以下两个原因：

1. JVM 低竞争状态下的同步消耗已经由于 JVM 1.5 版本后的优化降低很多，不需要应用层考虑太多争用同步消耗的问题
2. 语义上，双重检查锁定比内部类不易于理解。


## 为什么使用内部类静态变量可以实现单例模式？


1. JVM会保证类的静态变量只初始化一次，这保证了安全。
2. 内部类会在外部类第一次使用到它的时候才会被加载，这保证了懒加载。



## 为什么使用枚举类型可以实现单例模式


枚举类型实际上就是Enum类的子类，每一个枚举变量都是该类的一个静态对象，静态对象会由JVM保证只初始化一次。


## 如何破坏单例模式？


通过反射的方法获取到单例类的构造器方法，通过newInstance实例化一个对象出来。这个对象和原本的单例对象就不是同一个对象了，单例模式也就被破坏了。


## 观察者模式


## 装饰器模式


# Spring 相关

# 微服务开发相关

## 击穿，穿透，雪崩
- 击穿：热点 key 失效导致瞬间大量请求到 DB。可以加 DB 访问锁解决
- 穿透：特殊的 key（例如 null key）被大量请求，直接打到 DB。可以通过缓存特殊 key 解决
- 雪崩：同一时间大量缓存 key 失效，导致 DB 压力升高。可以通过缓存增加随机延迟避免同时失效大量 key

## 令牌桶实现？

# MQ

## 消息重复？

两个大方向：

1. 超时丢弃+消息id去重；
2. 语义上保证处理函数幂等

## 事务消息实现？

## 消息乱序处理？

# 其他

## 动态类型（dynamically typed）语言与静态语言（statically typed）区别？C++是哪种？其他语言呢？


区别：能否在编译期确定变量类型。C++、Java 为静态语言，Python、JavaScript 为动态语言。


## 强类型（strongly typed）语言与弱类型（weakly typed）语言的区别？有哪些例子？


区别：对隐式类型转换的容忍程度高则为弱类型语言。C++ 为弱类型语言，Python 为强类型语言。
