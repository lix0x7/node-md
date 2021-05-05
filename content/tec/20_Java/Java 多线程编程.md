# Java中的线程


- 进程和线程的区别
   - 进程时分配系统资源的最小单位，线程是分配时间片的最小单位
   - 一个进程可以拥有多个线程
   - 每个进程拥有自己的一整套数据，而隶属于同一进程的线程共享数据
   - 线程更轻量，创建、撤销一个线程比启动进程的开销要小得多
- ConcurrentModificationException
- Java实现线程
   - 实现Runnable接口，推荐
```java
public static void main(String args[]) {
   Runnable r = new Runnable() {
	  @Override
	  public void run() {
		 for (int i = 0; i < 100; ++i){
			System.out.println(i);
			try {
			   Thread.sleep(500);
			} catch (InterruptedException e) {
			   e.printStackTrace();
			}
		 }
	  }
   };
   new Thread(r).start();
}
```

   - 继承Thread类，不推荐
OOP的核心思想之一是面向接口编程，所以不推荐使用继承的方法。
```java
public class Core {
   public static void main(String args[]) {
	  CntThread cntThread = new CntThread();
	  cntThread.start();
   }
}
 
class CntThread extends Thread{
   public void run(){
	  for (int i = 0; i < 100; ++i){
		 System.out.println(i);
		 try {
			Thread.sleep(500);
		 } catch (InterruptedException e) {
			e.printStackTrace();
		 }
	  }
   }
}
```

   - 线程池，推荐
对于过多的需要独立线程任务，为每个任务创建一个线程会带来极大的开销，此时应该使用线程池。
- 没有可以强制线程终止的方法，interrupt方法可以请求终止线程
   - 对运行线程调用interrupt方法将会将线程的中断状态置位（true），这是一个boolean标志
      - 线程可以通过Thread.currentThread().isInterrupt()方法检测中断状态是否被置位
   - 在阻塞线程上调用interrupt方法，被阻塞线程会抛出InterruptException，并重置线程的中断状态位置为false，调用interrupt的线程会被该异常中断
   - [*]可以理解为，interrupt为字面意思，其他线程“打扰”了指定线程的执行，被“打扰”的线程可以接收到该打扰信号，至于作何反应，还是取决于被打扰的线程
- Java采取抢夺式线程调度
- 线程状态
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100510006-100e52cc-68fc-45ca-a7b7-da005de3885c.png#align=left&display=inline&height=570&margin=%5Bobject%20Object%5D&originHeight=570&originWidth=422&size=0&status=done&style=none&width=422)
   - Runnable状态译为“可运行”，Java中的可运行状态对应至传统的操作系统层面的线程状态，既可以是就绪状态，也可以是运行状态。
   - 当线程试图**获取锁**的时候，会进入Blocked状态；成功取得锁后，恢复至Runnable状态
   - 当线程需要等待其他线程的通知时，进入Waiting状态。
      - 要注意，Waiting与Blocked状态有很大不同
   - 当线程调用有计时参数的方法时，会进入Timed Waiting状态。例如Thread.Sleep()

![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100510193-bd5430cf-79b7-4905-ac19-895cba9c1082.png#align=left&display=inline&height=531&margin=%5Bobject%20Object%5D&originHeight=531&originWidth=778&size=0&status=done&style=none&width=778)

- 线程属性
   - 优先级
      - Java中默认有1~10个线程优先级级别，默认为5
      - 但是，虚拟机实现线程优先级是依赖于系统实现的。一般来讲，系统级的个数都会更少。甚至在Sun为Linux提供的JVM中，线程的优先级被忽略：所有的线程优先级始终相同。
      - 因为上述原因，不要过度依赖优先级，也不要将程序的正确性依赖于优先级。
      - 抢夺式调度可能会将低优先级线程饿死
   - 守护线程
      - 通过Thread#setDaemon(true)将线程实例设置为守护线程，在线程启动前调用
      - 只要当前JVM还有任何非守护线程没有停止工作，守护线程就会存在；当所有非守护线程结束后，守护线程和JVM一同停止工作，此时不保证守护线程执行完毕
      - 最典型的守护线程就是GC了



# 同步


- synchronized关键字
   - Java的每一个对象都有一个内部锁
   - 可重入锁
   - 不同的修饰对象
      - 修饰代码块，相当于对执行被修饰代码块的对象加锁
      - 修饰方法，相当于默认对于该方法构造了一个Reentrantlock，即对调用这个方法的对象加锁
      - 修饰静态方法，相当于对该类的所有实例加锁
      - 修饰类，相当于对所有该类的实例加锁
   - 在被声明为synchronized的方法中调用wait()、notify()、notifyAll()方法，相当于调用了默认锁的await()、signal()、signalAll()方法（）
   - Java Core不建议使用上述的Reentrantlock和synchronized关键字，它推荐使用java.util.concurrent包中的一种机制
- 锁对象
   - Reentrantlock 可重入锁
      - 示例
```java
lock.lock();
try{
	// do what you want
}catch (InterruptedException){
	// catch code
}finally {
	lock.unlock();
}
```

      - 可重入，指的是某一线程可以重复获得已经持有的锁。例如上述try代码块中的内容如果有一方法m()调用中再次调用了lock.lock()，该锁对象的持有计数器有1变为2。当方法m()退出时，该计数器变为1。当该try/catch块退出后执行finally块后，该计数器变为0，线程释放锁
      - 注意lock.unlock()一定要在finally块中，这样才能保证锁释放，其他线程不会永远阻塞
      - 使用该锁便不能使用try-with-resouces语法，因为该锁的释放方法为unlock()而非close()
      - 还有类似的tryLock()方法，可以在获取锁失败后去做别的事情。tryLock方法也可以添加参数，形如tryLock(100, TimeUnit.SECONDS)，其中TimeUnit为枚举类型。
   - 条件对象
      - Condition condition = lock.newCondition()，此处condition为一个条件对象
      - 在需要满足条件才能运行的代码段使用如下结构：
```java
lock.lock();
try{
	while (!whatYouWantToCheck){
		condition.await();
	}
	// do what you want
	condition.signalAll();
}catch (InterruptedException e){
	// catch code
} finally {
	lock.unlock();
}
```

         - 首先检查条件，如果不满足则始终调用条件await()阻塞当前线程并放弃锁，当前线程进入了该条件的等待集
         - 如果有某一线程调用条件对象的signalAll方法，则该条件对象的等待集中的线程阻塞状态都将解除，变为Runnable状态，随机通过竞争访问的方式访问所需对象。
         - 但是被解除阻塞状态的线程只是有可能满足条件，所以仍需使用while方法去检查指定条件，如果不满足，还是要继续被阻塞。
      - 条件对象还有一signal()方法，无参数，可以随机解除该条件对象等待集中的某个线程的阻塞状态。这比解除所有线程的阻塞状态要高效，但是如果被解除阻塞状态的线程发现未能满足条件而再次被阻塞，且没有其他线程调用signal方法，那么整个系统将陷入死锁。
   - Reentrantlock与synchronized的不同之处
      - 一个为类，一个为关键字
      - Reentrantlock等待可中断（trylock），可以设置多个条件（condition），可以实现公平锁（new ReentrantLock(true)）
- 监视器（Monitor）的概念
监视器是一种实现多线程编程的思路。监视器的特征如下：
   - 监视器是只包含私有域的类
   - 每个监视器类的对象有一个相关的锁
   - 使用上述锁对所有方法加锁，并且该过程是自动的。对于Java而言，使用synchronized声明的方法可以达到此效果
   - 该所可以有任意多个相关条件

一个Java对象可以看作一个Montior，但是在以下几方面有明显不同：

   - 不要求所有域私有
   - 不要求所有方法都使用synchronized修饰
   - 锁对象可以由程序访问到
- Volatile域
   - Volatile关键字为实例域的同步访问提供了一种免锁机制，但他不提供原子性，详情参见JVM相关笔记
- 死锁
   - 条件
      - 互斥
      - 请求与保持
      - 不剥夺
      - 循环等待
   - Java没有提供可以避免或打破死锁的手段，所以必须小心设计程序
- 线程局部变量
   - ThreadLocal对象使用initialValue方法定义首次调用get方法的生成的对象。如果一个给定线程中首次调用get方法，那么就会调用initialValue方法生成对象；之后调用get方法会返回属于当前线程的对象实例。
   - 实例如下，详见代码及注释
```java
public class ThreadLocalPractice {
public static void main(String[] args){
//    Runnable r = new TlRun();
	Runnable r = new Run(); // 与上面一行启用了不同的线程，区别在于是否使用了ThreadLocal保存类内部数组
	new Thread(r).start();
	new Thread(r).start();

	// 使用ThreadLocal的输出如下：
		// [Thread-0: 0, Thread-0: 1, Thread-0: 2, Thread-0: 3, Thread-0: 4]
		// [Thread-1: 0, Thread-1: 1, Thread-1: 2, Thread-1: 3, Thread-1: 4]
	// 未使用ThreadLocal的输出如下：
		// [Thread-0: 0, Thread-0: 1, Thread-0: 2, Thread-0: 3, Thread-0: 4]
		// [Thread-0: 0, Thread-0: 1, Thread-0: 2, Thread-0: 3, Thread-0: 4, Thread-1: 0, Thread-1: 1, Thread-1: 2, Thread-1: 3, Thread-1: 4]
	// 可以看到区别在于多个线程是否共享了同一个ArrayList<String>实例
}

}

// 使用了ThreadLocal的类
class TlRun implements Runnable{
	private ThreadLocal<ArrayList<String>> ths = new ThreadLocal<>(){
		// 重写initialValue方法，返回一个ArrayList<String>对象
		@Override
		protected ArrayList<String> initialValue() {
			return new ArrayList<String>();
		}
	};
	
	@Override
	public void run() {
		for (int i = 0; i < 5; ++i){
			ths.get().add(Thread.currentThread().getName() + ": " + i);
		}
	
		System.out.println(ths.get());
	}
}

// 未使用ThreadLocal的类
class Run implements Runnable{
	private ArrayList<String> ths = new ArrayList<>();
	
	@Override
	public void run() {
		for (int i = 0; i < 5; ++i){
			ths.add(Thread.currentThread().getName() + ": " + i);
		}
	
		System.out.println(ths);
	}
}
```

- 读写锁 ReentrantReadWriteLock，见下述示例代码
```java
// 创建读写锁
ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
// 获取读锁
Lock readLock = rwl.readLock();
// 获取写锁
Lock writeLock = rwl.writeLock();
```

   - 随后，对读取方法加读锁，对修改方法加写锁即可。这是一个读者-写者问题
- stop/suspend方法，这两个方法均已被弃用
   - stop方法由于是强制结束线程，所以可能导致线程未完全执行完毕就强制结束，导致对象状态被破坏（例如转钱的操作只执行了扣除源账户余额，而为加到目标账户上）。stop方法会释放线程锁持有的锁。
   - suspend方法在调用时会将指定进程立刻挂起，但不会释放该线程持有的锁。所以，如果suspend的调用线程需要获取同一个锁，那么程序会陷入死锁。
- java.util.concurrent.atomic



# 线程相关的数据结构


- 阻塞队列 BlockingQueue接口
   - 生产者-消费者模型
      - 生产者线程向队列插入元素，消费者线程取出元素。
      - 生产者线程在插入元素时如果队列已满，就会被阻塞
      - 消费者线程在取出元素时如果队列为空，就会被阻塞
   - 方法
      - put(E)
添加一个元素，如果队列满，则阻塞
      - take()
移出并返回头元素，如果队列空，则阻塞
      - offer()
添加一个元素并返回true，如果队列满，则返回false
      - peek()
返回队列头元素但不移出，如果队列空，则返回null
      - offer方法和poll方法还有带超时的变体，如下
         - boolean success = q.offer(x, 100, TimeUnit.MILLSECONDS)
         - Object head = q.poll(100, TimeUnit.MILLSECOND)
   - java.util.concurrent提供了几个阻塞队列的变种
      - LinkedBlockingQueue
      - LinkedBlockingDeque
      - ArrayBlcokingQueue
      - PriorityBlockingQueue
      - DelayQueue
- java.util.concurrent.ConcurrentHashMap
   - HashMap在并发环境下可能出现死锁
todo
   - JDK1.7及以前的实现
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100509909-468385bc-78ab-4f32-8f10-1aa726083369.png#align=left&display=inline&height=537&margin=%5Bobject%20Object%5D&originHeight=537&originWidth=502&size=0&status=done&style=none&width=502)
并发的散列映射表，可高效地支持大量的读者和一定数量的写者。默认情况下，假定可以有多达16个写者线程同时执行，如果有多余16个线程的写入，则多出来的线程将暂时阻塞。可以指定更大数目的构造器，但一般情况下不需要。
通过Segment(Table)->Bucket->Entry的结构实现，Segment继承自ReentrantLock，可以充当锁的角色。Entry链表的读写机制类似于CopyOnWrite容器。每个Entry的数据声明为volatile变量，这意味着JVM会保证数据变化是线程安全的。key/hash/next都被声明为final，这意味着已有的链表不会发生除头节点以外的结构性变化。
      - 结构性修改
         - put操作是在每一个链表的链头插入（这样不会影响既有读操作的进行）
         - clear操作将桶指向的Entry引用置空，但原Entry链表仍然存在，不影响既有的读取操作
         - remove操作，例如删除节点C
对节点C前的链表进行克隆，并R后续的部分链接在克隆的部分链表的末尾，如下图所示：
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100509897-a5af508b-8ecf-4bc0-a543-48b596e9ae47.png#align=left&display=inline&height=136&margin=%5Bobject%20Object%5D&originHeight=136&originWidth=570&size=0&status=done&style=none&width=570)

写操作时会对当前桶所对应的链表加锁（实际实现中是对链表的第一个节点加锁，为了不浪费空间去维护一个锁），但是这个锁不影响读操作，只影响其他写操作。特别的是，当一个桶的链表不存在时，该类是通过CAS的方式插入第一个节点的，这样可以避免加锁的开销。
该实现只保证了弱一致性，详情可参见IBM的[这篇文章](https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/index.html)。

   - JDK1.8及其以后的实现
todo
- copyOnWriteArrayList、copyOnWriteArraySet
   - 上述两个类使用了写时复制的技术，即在对其进行修改操作时，现将源数组复制一份到新数组，在新数组上进行写操作，在写操作完成后再将数组的引用指向该新数组。这个过程完全不会修改原数组，所以读取操作是可以不加锁的。但是多个写之间，还是需要加锁的
   - 这种处理方案的缺点就是，读操作并不是实时的，有可能落后于写操作，但保证了弱一致性
   - 在数据较多的情况下复制量比较大，对系统资源消耗很大
- 早期的线程安全集合
   - 可以使用Collections.synchronizedList这类的同步包装器将集合类变为线程安全的，但大多数情况下效率都比不过java.util.concurrent包中定义的集合。例外的是写操作比较多的数组，使用同步后的ArrayList要比CopyOnWriteArrayList效率更好
   - Vector
将ArrayList的所有方法使用synchronized修饰符声明
   - Hashtable
相比HashMap不支持null作为key或value
- Callable/Future
   - Callable接口与Runnable接口类似，不同之处是Callable是一个有泛型参数类型T返回值的Runnable，并且方法名有所不同，取代了Runnable方法中的run方法，改为call，并返回类型为T的返回值
   - Future是用于保存异步运算结果的接口，它使用get方法获取计算结果，如果没有计算完成，则被阻塞至计算完成；cancel方法用于中断计算；isDone和isCancelled方法分别用来查看是否计算完成和取消
   - 使用FutureTask包装器将Callable转换为Future和Runnable，它同时实现了二者的接口，可以传给线程运行，也可以调用Future接口的方法查看运行结果
   - 示例如下：
```java
public static void main(String[] args){
   Callable<Integer> callable = new Callable<Integer>() {
	  @Override
	  public Integer call() throws Exception {
		 // things you want to do
		 // and return the result
	  }
   };
   FutureTask<Integer> futureTask = new FutureTask<>(callable);
   new Thread(futureTask).start();

   try {
	  Integer ret = futureTask.get(); // 获取计算结果，如果尚未生成，则阻塞
   } catch (InterruptedException e) {
	  e.printStackTrace();
   } catch (ExecutionException e) {
	  e.printStackTrace();
   }
}
```


# 执行器与线程池


- 如果程序中创建了很多生命期很短的线程，应该使用线程池。一个线程池中包含很多创建好的线程，将Runnable对象交给线程池，线程池就会将Runnable对象分配给一个特定的线程并执行。当run方法退出时，线程不会死亡，而是在池中等待为下一个请求提供服务
- 执行器（Executors）类有许多静态工厂方法来构建线程池（ExecutorService）对象，如下：
   - newCachedThreadPool，必要时创建新线程，空闲线程保留60s;
   - newFixedThreadPool，包含固定数量的线程，空闲线程会一直保留
   - newScheduledThreadPool，预定执行，可以设置在指定延迟之后执行，或者周期性的运行，或者结合上述两者功能的组合
- 使用submit方法提交Runnable或Callable对象实例
- 使用shutdown方法，会在所有任务结束后关闭线程池，否则整个程序无法运行结束
- 线程池代码示例：
```java
public class ThreadPoolPractice {
	public static void main(String[] args) throws ExecutionException, InterruptedException {
		ExecutorService pool = Executors.newCachedThreadPool(); // 申请线程池
		List<Future<Integer>> results = new ArrayList<>();  // 保存每个线程的计算结果
		int total = 0;  // 保存所有返回结果之和

		for (int i = 0; i < 5; ++i){
			// 创建五个线程并提交(submit)至线程池运行
			Callable<Integer> c = new Callable<>() {
			@Override
			public Integer call() throws Exception {
				int cnt = 0;
				for (; cnt < 4; ++cnt){
					System.out.println(Thread.currentThread().getName() + ": " + cnt);
				}
				return cnt;
			}
			};
			Future future = pool.submit(c);
			results.add(future);
		}
		// 迭代results取出每个线程返回值并打印
		for (Future<Integer> f: results){
			total += f.get();
			System.out.println("Thread end, and the total val now is " + total);
		}
		// 关闭线程池
		pool.shutdown();
	}
}
```

- ExecutorService#invokeAny|invokeAll
   - invokeAny(Collection tasks)，运行tasks中的所有任务，并返回最先完成的任务的结果
   - invokeAll(Collection tasks)，运行tasks中的所有任务，并返回所有任务结果（Future）的集合
- Fork-Join框架
   - 实现RecursiveTask或者RecursiveAction，上述两者区别在于前者返回一个T类型的返回值，而后者无返回值，均为抽象类
   - 主要方法
      - ForJoinPool#invoke
         - 看源码可以发现，该方法对于调用RecursiveTask、RecursiveAction时与submit方法相同
      - RecursiveTask|RecursiveAction#invokeAll(Collection)
         - 在RecursiveTask、RecursiveAction内部使用该方法，开始调度执行所有计算任务
      - RecursiveTask#join，获取计算任务的返回值，如果未完成计算，则阻塞
   - 下述代码以RecursiveTask为例，展示了一个计算文件夹下所有文件数目的任务
```java
public class RecursiveTaskPractice {
   public static void main(String[] args){
	  ForkJoinPool pool = new ForkJoinPool();
	  SearchTask searchTask = new SearchTask(new File("C:\\Files\\Project"));
	  pool.invoke(searchTask);
	  pool.shutdown();
   }
}

class SearchTask extends RecursiveTask<Integer> {
   private File f;

   public SearchTask(File f){
	  this.f = f;
   }

   @Override
   protected Integer compute() {
	  int cnt = 0;    // 记录每个文件夹下的文件数量，包含子文件夹下的文件数量
	  File[] files = f.listFiles();
	  for (File file: files){
		 if (file.isDirectory()){
			SearchTask st = new SearchTask(file);
			invokeAll(st);
			cnt += st.join();
		 }else {
			++cnt;
//          System.out.println(file.getAbsolutePath());
		 }
	  }
	  System.out.println(Thread.currentThread().getName() + " : " + f.getAbsolutePath() + " : " + cnt);
	  return cnt;
   }
}
```


# 同步器


- CyclicBarrier
可重用，Cyclic意即循环，通过reset()方法重置
```java
public class CyclicBarrierPractice {
	public static void main(String[] args) throws BrokenBarrierException, InterruptedException {
		CyclicBarrier cb = new CyclicBarrier(2);
		new Thread(new Runnable() {
			@Override
			public void run() {
			try {
				System.out.println(Thread.currentThread().getName() + " has began");
				Thread.sleep(1000);
				// 使当前进程等待直到在该CyclicBarrier上等待的线程数达到其设置的阈值
				cb.await();
				System.out.println(Thread.currentThread().getName() + " has came");
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				e.printStackTrace();
			}
			}
		}).start();
		// 使当前进程等待直到在该CyclicBarrier上等待的线程数达到其设置的阈值
		cb.await();
		// 此时主线程和我们创建的线程都到达该CyclicBarrier
		System.out.println("main has came");
	}
}
```

- CountDownLatch
不可重用
```java
public class CountDownLatchPractice {
	public static void main(String[] args) throws InterruptedException {
		// 创建Latch对象并初始化计数值为5
		CountDownLatch latch = new CountDownLatch(5);
		new Thread(new CDLP(latch)).start();
		// 等待计数归零
		latch.await();
		System.out.println("latch over");

		// 输出结果 1,2,3,4,5,"latch over",6,7,...,20
	}
}

class CDLP implements Runnable{
	CountDownLatch latch;
	CDLP(CountDownLatch latch){
		this.latch = latch;
	}
	@Override
	public void run() {
		for (int i = 0; i < 20; ++i){
			try {
				// 每循环一次技术减一
				latch.countDown();
				System.out.println(i);
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

- CAS
