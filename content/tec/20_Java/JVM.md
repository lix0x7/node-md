# Ref


- 深入理解Java虚拟机（第2版）



# JVM内存区域、自动内存管理机制


- JVM所管理的内存包含以下运行时数据区
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100394973-e04b3770-4472-4086-ba07-9c0854e3d93a.png#align=left&display=inline&height=373&margin=%5Bobject%20Object%5D&originHeight=373&originWidth=477&size=0&status=done&style=none&width=477)
   - PC
      - 每个线程有自己独立的PC，称这类内存为“线程私有”内存
      - 如果线程执行的Java方法，那么PC记录的是虚拟机字节码地址；如果执行的为Native方法，则这个计数器为空（Undefined）
      - 此内存区域是唯一一个在JVM规范中没有规定任何OutOfMemoryError情况的区域
   - 虚拟机栈（VM Stack）
      - 线程私有
      - 生命周期与线程相同
      - 虚拟机栈描述的是Java方法执行的内存模型，主要存储的是方法执行时的栈帧（Stack Frame），用于存储局部变量表、操作数栈、动态链接、返回地址等信息
         - 局部变量表
            - 局部变量表存储了编译器可知的基本数据类型、引用类型（reference指针）、returnAddress类型（返回地址）
            - long和double类型占用2个局部变量空间（Slot），其他类型均为1个。
            - 局部变量表在编译期间完成分配，是完全确定的，运行期间不会再改变。
            - 会抛出StackOverflowError（请求深度超出JVM规定的上限）和OutOfMemoryError
   - 本地方法栈（Native Method Stack）
      - 线程私有
      - 功能与虚拟机栈类似，区别在于本地方法栈是为Native方法服务的
      - JVM规范对该部分没有规定，由具体虚拟机自行实现
      - 会抛出StackOverflowError（请求深度超出JVM规定的上限）和OutOfMemoryError
   - 直接内存（Direct Memory）
      - 线程私有
      - 并不是JVM的区域，而是物理机的内存区域
      - 通过NIO类使用Channel和Buffer的I/O方式，使用Native函数库直接分配堆外内存，然后通过储存在Java堆中的DirectByteBuffer引用这块内存对该内存进行访问。这样用来增加性能，避免了在堆和内存间频繁地复制数据
   - 堆（Heap）
      - 堆被所有线程共享
      - 虚拟机启动时创建
      - Java规范描述：所有对象实例及数组都要在堆上分配。但是JIT和其他优化技术的发展导致了对象也不一定全部在堆上分配，但绝大多数对象还是的。
      - 垃圾回收的主要区域
   - 方法区（Method Area）
      - 线程共享区域
      - 用于存储类信息、常量、静态变量、JIT编译后的代码
         - String类所指向的字符串常量都位于该位置
      - 运行时常量池，方法区的一部分
         - 存储编译期生成的Class文件中各种字面量和符号引用



## HotSpot JVM 中的对象


- 普通对象的创建（不包括数组或Class对象）
   - JVM遇到new指令时，先在常量池中定位该类，查看指定类是否已经被加载、解析和初始化，否则先执行类加载过程
   - 类加载检查通过后，在堆的闲置空间中分配空间
      - 分配算法
         - 指针碰撞（连续分配，没有碎片）：一般是带Compact过程的GC是使用，例如Serial、ParNew
         - 空闲列表（有碎片）：一般用于mark-sweep算法的GC，例如CMS
      - 线程安全问题：使用指针碰撞方法可能遇到多个线程同时读写指针，此时如何保证线程安全？
         - CAS+失败重试保证操作原子性
         - 本地线程分配缓冲（Thread Local Allocation Buffer, TLAB）：每个线程自己有一小块内存，称为TLAB，哪个线程需要分配空间，就现在自己的TLAB中分配，只有TLAB用完并分配新的TLAB时，才同步锁定TLAB
      - 分配完内存后，初始化内存空间为零值（不包括对象头）
      - 设置对象头，例如该对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、GC分代年龄
      - 将对象引用入栈
句柄引用（句柄池）or直接指针
   - 对象内存区域
      - 对象头（Header），一般是32bit的bitmap，64位机器上为64bit的bitmap
      - 实例数据（Instance Data）
      - 对齐填充（Padding）



# 垃圾回收机制与内存分配策略


- 概述
   - 关注的问题：回收哪些内存？何时回收？如何回收？
   - 垃圾回收涉及的是堆和方法区的内存空间，其他空间随生命周期都伴随着其线程的生命周期
- 判断对象是否已死的两种方法
   - 引用计数法（JVM未采用）
      - 有引用计数+1，无引用计数-1
      - 主流JVM未采用这一方法，引用计数法难以解决对象间相互循环引用的问题，会造成内存泄漏
   - 可达性分析（JVM采用）
      - 通过一系列称为GC Roots对象作为起点向下搜索，搜过的位置称为引用链。当一个对象和GC Roots之间没有任何引用链（两个节点不可达）时，证明此对象是不可用的。
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100395295-f5967a4b-3dab-4b62-bd3e-eee8853941e1.png#align=left&display=inline&height=484&margin=%5Bobject%20Object%5D&originHeight=484&originWidth=706&size=0&status=done&style=none&width=706)
      - 常被用作GC Roots的对象[*可以发现这些作为GC Roots的对象的生命周期都是与他们所属线程、方法的生命周期相同的]
         - 虚拟机栈帧中本地变量表引用的对象
         - 方法区中类静态属性引用的对象
         - 方法区中常量引用的对象
         - 本地方法栈中Native方法引用的对象
   - 引用级别
      - 直接通过给定地址的方式定义引用太过绝对（JDK1.2之前都是这样做的），对于缓存场景的情况，不能充分的利用GC回收内存
         - 缓存场景：当内存空间充足时，保留对象；当空间不足时，自动释放
      - JDK1.2之后，Java将引用扩充为如下四中：
         - 强引用（Strong Reference）
最常见，GC永远不会回收强引用的对象
         - 软引用（Soft Reference）
仅次于强引用，在将要发生内存溢出前会将软引用对象列入回收范围，如果仍然未回收到足够的内存空间，再抛出异常
         - 弱引用（Weak Reference）
类似软引用，但是弱引用对象只能生存至下一次GC之前，当GC开始，弱引用对象就会被回收
         - 虚引用（Phantom Reference）
虚引用不影响其GC，也不能通过虚引用取得对象；为对象设置虚引用的唯一目的就是能在对象被回收时收到一个系统通知
   - 回收过程中的两次标记与finalize()方法的执行
      - 在进行可达性分析后，没有与GC Roots相联的对象会被第一次标记
      - 被标记对象进行一次筛选，筛选条件为：是否覆盖了finalize方法？是否VM已经执行了finalize方法？
      - 当对象有必要执行finalize方法时，那么对象就会被添加到F-Queue队列中，由VM建立的Finalizer线程（低优先级线程）执行对象的finalize方法。**该线程不保证finalize方法运行结束**，因为GC要保证GC不会因为某一对象的finalize方法执行缓慢或死循环导致整个GC系统崩溃
      - GC对F-Queue中的对象进行第二次标记，如果对象在finalize方法执行过程中成功重新建立了引用关系，那么GC将该对象移除“即将回收”的集合
      - 上述自救机会只有一次，一个对象的finalize方法只会被执行一次，第二次GC时无法通过finalize方法自救
      - 不推荐使用finalize方法，代价高昂，而且不确定性大。
   - 回收方法区
      - 无用常量：无引用字符串常量
      - 无用类
         - 该类的所有实例都已经被回收
         - 加载该类的ClassLoader已经被回收
         - 该类对应的java.lang.Class对象没有在任何地方引用，无法在任何位置通过引用访问该类的方法
      - 在大量使用反射、动态代理、频繁自定义ClassLoader的场景下，回收无用类，具有类卸载功能很重要，防止方法区移除
   - 垃圾收集算法
      - 标记-清除（Mark-sweep）
         - 过程在前文的讲finalize时已经详细说明
         - 是后续算法的基础，其它算法也都是在改进此算法的不足
         - 缺点
            - 标记和清除的过程效率都不高
            - 会产生大量的外部碎片
      - 复制算法（回收新生代）
         - 将内存区域分成两部分（可以不均分）在标记结束以后，将不需要回收的对象复制到内存中的另一块区域，然后移动堆指针，再把之前的区域整体清除即可。
         - 现代商用JVM都使用这种算法回收**新生代**，由于新生代对象大多生命周期很短（IBM研究表明这个值在98%附近），所以这个区域的比例无需太过平均。具体如下：
            - 将内存分为一块较大的Eden空间和两块较小的Survivor空间，HotSpot默认Eden:Survivor大小比例为8:1。每次使用Eden和其中一块Survivor，当回收时，将Eden和Survivor中还存活的对象复制到另外一块Survivor上，然后清理掉Eden和上一次用过Survivor。
               - 这种情况下浪费的空间仅为10%
               - 当Survivor空间不足时，需要依赖老年代内存进行分配担保（Handle Promotion）
      - 标记-整理算法（Mark-Compact）（回收老年代）
先标记，然后将存活对象移至一端，再清理尾端边界外的内存，去除了内部碎片
      - 分代收集算法（综合上述两种）
分为新生代和老年代，然后分别应用上述两种算法
- HotSpot中的算法实现
   1. 枚举根节点，可达性分析
      - 枚举和分析过程中，所有的运行线程都要中断，以保证分析过程中不会出现引用关系变动。
      - OopMap（准确型GC普遍采用的一种哈希表，用来存储引用到地址偏移量的映射信息）
         - JVM通过遍历类信息（方法区）中OopMap可以直接得知所有可以作为GC Roots的对象的引用位置，该OopMap是在类加载过程中便计算好的（准确式GC）。
         - 对于栈上的局部变量引用，JVM会在一定的检查点（Safe Point）生成OopMap，避免对每条指令都生成OopMap造成过大的空间开销，也很好地利用了OopMap的优势。这些检查点一般有如下这些位置：
            - 循环末尾
            - 方法返回前、调用方法call指令后
            - 可能抛出异常的位置
         - 由于只有检查点位置的OopMap才是准确的，所以GC只会在检查点位置触发。
      - [[*引申]](http://rednaxelafx.iteye.com/blog/1044951)相对于OopMap，保守型GC只存储了对象所在的地址，而完全没有其他信息，GC在工作时需要去遍历运行栈和方法区检查每个位置是否为引用，而且这种检查充满了不确定性（只通过判断上下界、对齐条件等排除不是引用的数据，剩下的都认为是疑似引用），并不能确定该值是否为真的引用还是疑似引用（例如原始类型的数值等）。所以会造成如下问题：
      - 由于不知道疑似指针是否为真正的指针，所以其值不能改写，这也就意味着对应地址的对象不能移动位置，GC也就只能使用mark-sweep算法，浪费空间且效率不高
      - 堆内会出现一些已经死掉的对象，但是仍有疑似引用（实际上可能只是基本类型的存储值）指向他们，导致这些对象不能被回收
   2. 使所有线程运行至安全点中断或安全区内运行（这段时间称作`stop the world`），然后开始可达性分析与GC
      - 主动式中断
在安全点的位置插入标志，通过预先设定异常处理机制，到达该标志的线程将主动中断。
      - 安全区（Safe Region）
      - 在一段代码区间内，未发生引用变化，则称这段区间为安全区。
      - 比如，对于Sleep这种未到达检查点但十分耗时的操作，GC如果等待它执行完毕到达检查点将耗费太多时间，此时只需要该线程在安全区内，即可启动GC，而无需到达检查点。
      - 在线程要离开安全区时检查GC是否完毕，如果完毕则正常继续执行，否则则等待至GC完毕才可离开
   3. 垃圾收集器运行
收集器不止一种，并且可以搭配使用。本节只记录CMS（Concurrent Mark Sweep）/G1/ZGC收集器，其他的见下面表格
| 收集范围 | 算法 | 执行类型 |
| --- | --- | --- |
| ial | 新生代 | 复制 |
| ial Old | 老年代 | **标记整理** |
| New | 新生代 | 复制 |
| allel | 新生代 | 复制 |
| allel Old | 老年代 | **标记整理** |
| 老年代 | 标记清除 | 多线程并发 |
| 新生代+老年代 | 复制+**标记整理** | 多线程 |

      - `Serial`/`Serial Old`都会完全`stop the world`然后单线程进行GC再恢复用户线程
      - `ParNew`/`Parallel Old`也会完全`stop the world`，他们和Serial GC区别就是将GC过程变成多线程的。虽然叫Parallel，但并不是只得和用户线程并行，而只是自身的多个GC线程并行
- 三种垃圾收集器
   - CMS（Concurrent **Mark Sweep**）
      - 工作在老年代
      - 目标是使回收停顿时间最短
      - 基于Mark Sweep
      - 四个步骤，其中1,3过程需要中断用户进程：
         1. 初始标记（Initial Mark）中断
标记作为GC Roots及其能直接关联到的对象
         1. 并发标记（Concurrent Mark）并发
         1. 重新标记（Remark）中断
主要是重新标记在并发标记阶段出现标记变动的对象
         1. 并发清除（Concurrent Sweep）并发

![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100395164-9a06a137-19de-4c3e-a5e5-0ebd1a5c388c.png#align=left&display=inline&height=245&margin=%5Bobject%20Object%5D&originHeight=245&originWidth=892&size=0&status=done&style=none&width=892)

      - 缺点：
         - 对CPU资源敏感，在原本CPU资源不充裕的机器上还要分出运算能力去执行GC，会显著降低用户线程的运行速度
         - 无法处理浮动垃圾，即在步骤4中用户线程新产生的垃圾。
            - 这种情况下，收集器需要在老年代仍有剩余空间时便被触发，JDK1.5默认为68%，JDK1.6默认为92%
            - 若在GC过程中预留空间不足，则触发Concurrent Mode Failure，此时临时启用Serial Old收集器对老年区进行垃圾回收工作
         - 由于使用的是Mark Sweep，无法避免**外部碎片**，会在没有足够空间时触发Full GC，进行压缩，导致更长的停顿时间
   - G1（Garbage First）收集器
      - JDK1.5加入实验版本，JDK1.7正式发布商用版本
      - 同时工作在新生代和老年代
      - 整体看来基于**Mark-Compact**，局部region之间基于“复制”算法
      - 目标是使降低`stop the world`的时间，并且实现可预测的停顿：通过优先回收垃圾多的region实现
      - 堆布局与其他收集器不同
         - G1将整个堆划分为很多大小相等的区域（Region），同时保留了新生代和老年代的概念，但他们在物理上不再是隔离的了，他们都是一部分Region的集合
         - G1跟踪每个Region中的垃圾比例，维护了一个优先队列，每次根据允许的收集时间，优先回收时间允许范围内垃圾最多的Region。这种方法既保证了可预测停顿，又保证了回收的高效
         - 如何追踪跨Region引用？使用Remembered Set
每个Region有对应的Remembered Set，JVM发现对引用类型进行写操作时，生成一个Write Barrier中断，然后检查新写入的引用是否位于同一Region（对于传统的老年代新生代分区，该处则是检查老年代对象是否引用了新生代对象），如果引用了自身Region外的对象，便这条引用信息记录到**被引用对象**所在Region的Remembered Set中。在GC时，将该Region内的对象纳进GC Roots的枚举范围便可以保证即使不对所有Region进行扫描，也不会遗漏引用信息。
      - 运行过程（不考虑维护Remembered Set）：
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100394993-24c148f5-5648-481b-9489-59e917b589e7.png#align=left&display=inline&height=182&margin=%5Bobject%20Object%5D&originHeight=182&originWidth=849&size=0&status=done&style=none&width=849)
前三个阶段所做的事情和CMS收集器做的事情基本相同，相同的地方不再赘述，下面只记录不同的地方
         - 初始标记（中断）
修改TAMS（Next Top at Mark Start）值，让下一阶段的用户线程知道最新的可用Region位置
         - 并发标记（并发）
将此处产生的需要存储在Remembered Set中的数据暂存在Remembered Set Logs中
         - 最终标记（中断）
将Remembered Set Logs中的数据合并到Remembered Set中
         - 筛选回收（中断）
对各个Region的回收价值及成本进行排序，制定回收计划然后回收
   - ZGC(java11)
todo
- 内存分配与回收策略
   - 下图为堆区的内存组织，由上到下依次为：

![](https://cdn.nlark.com/yuque/0/2019/jpeg/657413/1577100395268-6d722030-a1af-4bdd-b4cf-d33018e8b4bd.jpeg#align=left&display=inline&height=283&margin=%5Bobject%20Object%5D&originHeight=283&originWidth=515&size=0&status=done&style=none&width=515)

   - 新生代
从左到右依次为一块Eden、两块Survivor（From Space、To Space）
   - 老年代
   - 永久代（HotSpot VM中的概念，指的是方法区）
   - 几种不同的GC
针对HotSpot VM的实现，它里面的GC其实准确分类有以下几种：
      - Partial GC：并不收集整个GC堆的模式
         - Young GC(Minor GC)：只收集young gen的GC
         - Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
         - Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
      - Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。
      - [*]还有称为Major GC的，但有时指Old GC，也有时指Full GC
   - 内存分配策略
      - 小型对象在Eden分配空间，若空间不足，触发Minor GC。
      - 大型对象在老年代分配空间
      - 新生代对象每经历一次GC未被回收则年龄+1，当年龄到达阈值（默认为15），则晋升至老年代
年龄动态判定：当Survivor中的相同年龄对象所占空间的总和大于Survivor空间的一半，那么都等于该年龄的对象可以直接进入老年代，而无视年龄阈值（也可以用这个解释大型对象直接在老年代划分空间的情况）
   - 在Minor GC前，VM需要先检查老年代剩余空间大小是否足够容纳新生代所有对象，这是为了避免极端情况下所有对象都不能被回收且另一个Survivor区不足以容纳所有对象，而需要经过空间分配担保使新生代对象直接进入老年代的情况。
      - 如果足够，进行Minor GC
      - 如果不足够，进行Full GC



# 性能监控、故障处理


# 调优


# 虚拟机执行子系统


## 类文件结构


- [*]`Javac->.class file->Java Runtime Env`这个过程和`Language->LLVM AST->LLVM->X86`这个过程有着异趣同工之妙，通过标准的中间表示（.class或AST）和多平台的执行/编译环境（JVM或LLVM编译器）运行或生成了平台无关代码。虽然一个是在JVM上运行，一个是通过不同的LLVM编译到不同的平台汇编。
- 类文件结构
   - Wikipedia列出的表格很好，参看https://en.wikipedia.org/wiki/Java_class_file
   - 使用`javap -verbose *.class`可以方便的查看
   - 以字节为基础的二进制流
   - 类文件中只有无符号数和表的概念：其中u1/u2/u4/u8分别代表1、2、4、8字节的无符号数；表是有无符号数和表构成的复合数据结构
   - 按字节顺序介绍
      - 0-3：class文件根据文件头的前4个字节（`0xCAFEBABE`）作为身份识别的标志，而不是文件后缀名
      - 4-7：5-6为次版本号，7-8为主版本号。JVM兼容低版本号class文件但拒绝执行高版本号文件
      - 8-~：常量池
         - 常量池中的每一个常量都是一个表结构，共有14种
         - 用于存储字面量（Literal）和符号引用（Symbolic Reference），
            - 字面量包括方法名、类名、接口名、文本字符串、声明为final的常量、类静态变量等，类文件中通过索引查找这些字面量
            - 符号引用属于编译原理的概念，包含如下三类常量：类和接口的全限定名，字段的名称和描述符、方法的名称和描述符
         - 0-1：u2类型数据，标志常量池大小
         - 1-~：常量的集合
每一个常量的表结构开始部分都是一个u1类型的标志位，标志该常量类型，具体见下图；这14种常量又各自有自己的表结构，此处不再赘述
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100395027-2eab23fb-153c-4561-a59e-b999967ccd7d.png#align=left&display=inline&height=474&margin=%5Bobject%20Object%5D&originHeight=474&originWidth=818&size=0&status=done&style=none&width=818)
      - 访问标志
2字节，代表当前类或接口层次的访问信息。如public/static/final/abstract等
      - 类索引、父类索引与接口索引集合，通过索引查询常量池，确定具体的名称字符串
         - 类索引：u2，确定类的全限定名
         - 父类索引：u2，确定父类的全限定名
         - 接口索引集合：u2+n*u2，确定一串实现接口的名称，按java源码中的顺序依次排列。前一个u2代表接口总数n。
      - 字段表、方法表（两部分，但很类似，因此写在一起）
         - 用于描述声明的变量、方法
         - 只包含类本身，不包含父类中的内容
         - 依次为访问限制标志位、名称索引、描述符索引和属性计数、属性列表
            - 描述符
               - 用于描述字段的数据类型、方法的参数列表（数量、类型、顺序）、返回值
               - 基本数据类型和void的描述符为其首字母大写，对象类型使用L加对象的全限定名表示，具体见下图
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100395112-9b3f90e7-d05e-41aa-9b7a-6bf49b212f58.png#align=left&display=inline&height=214&margin=%5Bobject%20Object%5D&originHeight=214&originWidth=837&size=0&status=done&style=none&width=837)
               - 数组类型每一维将以一个`[`表示，例如定义`java.lang.String[][]`，其描述符为`[[Ljava.lang.String`
               - 描述符描述方法时，将返回值后置，并将参数列表用小括号括起来，例如方法`int					 indexOf(char[] source, int sourceOffset, int sourceCount, char[]					 target, int targetOffset, int targetCount, int fromIndex)`的描述为`([CII[CIII)I`
         - 方法表中有可能出现编译器自动添加的类构造器`<clnit>`（对于**静态变量**等进行初始化）和实例构造器`<init>`（对于**实例属性**等进行初始化）
      - 属性表集合，存储类的附加信息，不单独存在于类文件中，而上述每一部分都可以有自己的属性表，所以放到此处一起讨论，所有的属性表类型详见下面表格。对于每个属性，使用一个u2提供属性表名称索引，使用一个u4表明属性表长度，随后填写自定义属性即可
| 使用位置 | 含义 |
| --- | --- |
| 方法表 | Java代码编译成的字节码指令 |
| antValue | 字段表 |
| cated | 类，方法，字段表 |
| tions | 方法表 |
| singMethod | 类文件 |
| Class | 类文件 |
| umberTable | Code属性 |
| VariableTable | Code属性 |
| MapTable | Code属性 |
| ture | 类，方法表，字段表 |
| eFile | 类文件 |
| eDebugExtension | 类文件 |
| etic | 类，方法表，字段表 |
| VariableTypeTable | 类 |
| meVisibleAnnotations | 类，方法表，字段表 |
| meInvisibleAnnotations | 表，方法表，字段表 |
| meVisibleParameterAnnotation | 方法表 |
| meInvisibleParameterAnnotation | 方法表 |
| ationDefault | 方法表 |
| trapMethods | 类文件 |

         - Code，方法表中的属性表
            - 最重要的一种属性表，存储具体方法体中代码编译后的字节码等信息，具体字段如下：
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100394844-5a5d5b03-a630-4a08-8969-aefca15aa75f.png#align=left&display=inline&height=362&margin=%5Bobject%20Object%5D&originHeight=362&originWidth=819&size=0&status=done&style=none&width=819)
            - 对于有的方法，Code属性表还会有异常表子表
如下源代码：
```java
public class TryCatchBytescode{
	public static void main(){
	}
	public int m(){
		try{
			return 1;
		}catch(Exception e){
			return 2;
		}finally{
			return 3;
		}
	}
}
```

            - 编译为.class文件后，再使用javap可以看到如下方法表及其Code属性表和Exception属性表，双斜杠注释是自己补充的说明：
```java
public int m();
	descriptor: ()I
	flags: (0x0001) ACC_PUBLIC
	Code:
	stack=1, locals=4, args_size=1
		0: iconst_1 //将int(1)推至栈顶
		1: istore_1 //将栈顶int型数值存入第二个本地变量
		2: iconst_3 //将int(3)推至栈顶
		3: ireturn	 //从栈顶弹出一个int数值，函数返回，将该返回值压栈给调用者
		4: astore_1 //将栈顶引用对象存入第二个本地变量，此处指的是异常对象
		5: iconst_2 //将int(2)推至栈顶
		6: istore_2 //将栈顶int型数值存入第三个本地变量
		7: iconst_3 //将int(3)推至栈顶
		8: ireturn	 //从栈顶弹出一个int数值，函数返回，将该返回值压栈给调用者
		9: astore_3 //将栈顶引用数值存入第四个本地变量
		10: iconst_3 //将int(3)推至栈顶
		11: ireturn	 //从栈顶弹出一个int数值，函数返回，将该返回值压栈给调用者
	Exception table:
		from	to	target type
			0	 2	 4	 Class java/lang/Exception
			0	 2	 9	 any
			4	 7	 9	 any
	LineNumberTable:
		line 6: 0
		line 10: 2
		line 7: 4
		line 8: 5
		line 10: 7
	StackMapTable: number_of_entries = 2
		frame_type = 68 /* same_locals_1_stack_item */
		stack = [ class java/lang/Exception ]
		frame_type = 68 /* same_locals_1_stack_item */
		stack = [ class java/lang/Throwable ]
```

            - 
可以发现，对于finally的处理就是冗余生成，多次复制了"return 3;"这条语句的字节码。Exception Table的含义如下：
               - 0-2行间的代码间如果出现了Exception类型的异常则跳转至第4行
               - 0-2行间的代码如果没有问题就转至第9行的finally语句处理
               - 4-7行间的代码（异常处理部分的代码）如果出现异常（异常内部再次异常），则跳转至第9行finally处理
- 字节码介绍
   - 字节码操作中原始类型相关的指令都会以类型首字母作为开头，例如istore、iload、ireturn、dconst、dload；引用类型相关的指令以a开头，例如aconst、aload、areturn等。后续所有指令开头的T均代表类型标志。
   - 由于只有double和long占用栈上的两个slot，其他类型都占用一个slot，所以对于byte和short，JVM会将他们符号扩展为相应的int类型数据；对于boolean和char类型，会零扩展为相应的int类型数据。所以在JVM中，大部分对于byte、short、boolean、char类型实际上都是使用int类型作为运算类型的
   - 加载、存储指令
      - 将一个操作数加载至操作数栈：`Tload`、`Tload_<n>`
此处n允许取指为0-3，这是一个java的过早优化，但是加入JIT后已经没有意义，等同于没有下划线的版本。[详情见此知乎回答](https://www.zhihu.com/question/54390587)
      - 将数值从操作数栈取至局部变量表：`Tstore`、`Tstore_<n>`
      - 将一个常量加载至操作数栈：`bipush`、`sipush`、`ldc`、`aconst_null`、`Tconst_<i>`
      - 扩充局部访问表索引的指令：`wide`
   - 运算指令
   - 类型转换指令
      - 默认支持宽化类型转换：
byte、char、short -> int -> long -> float -> double
      - 窄化类型转换必须使用显示指令
i2b、i2c、i2s、l2i、f2i、d2i…
   - 对象创建与访问指令
      - 创建类实例：new
      - 创建数组指令：newarray、anewarray、multianewarray
      - 访问类字段（static）和实例字段（non static）：getstatic、putstatic、getfield、putfield
      - 将数组元素加载至栈顶：Taload
      - 将栈顶值到数组元素：Tastore
      - 取数组长度：arraylength
      - 检查类实例类型：instanceof、checkcast
   - 操作数栈管理指令
      - 将1个或2个元素出栈：pop1、pop2
      - 复制栈顶值并压栈到栈顶：dup
      - 栈顶两个元素交换顺序：swap
   - 控制转移指令
      - 条件分支：if*、ifnull、ifnonnull
      - 复合条件分支：tableswitch、lookupswitch
      - 无条件分支：goto、jsr、ret
   - 方法调用和返回指令
invoke*
   - 异常处理指令
athrow
   - 同步指令
JVM通过管程（Monitor）[?]实现方法同步，对应的指令有：monitorenter、monitorexit
例如，对于如下方法



```java
public int m(Object f){
	synchronized (f){
		// code
		return 1;
	}
}
```


   - 将会被翻译成如下字节码



```java
public int m(java.lang.Object);
descriptor: (Ljava/lang/Object;)I
flags: (0x0001) ACC_PUBLIC
Code:
	stack=2, locals=4, args_size=2
	0: aload_1
	1: dup
	2: astore_2
	3: monitorenter
	4: iconst_1
	5: aload_2
	6: monitorexit
	7: ireturn
	8: astore_3
	9: aload_2
	10: monitorexit
	11: aload_3
	12: athrow
	Exception table:
	from    to  target type
		4     7     8   any
		8    11     8   any
	LineNumberTable:
	line 11: 0
	line 13: 4
	line 14: 8
```


   - 可以注意到，编译器通过自动产生了一个异常处理器。通过异常表的方式，JVM保证了无论如何monitorexit都会被执行



## 类加载机制


- 类的整个生命周期包括：加载、验证、准备、解析、初始化、使用、卸载，共7个阶段，顺序如下，其中验证、准备、解析统称为连接
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100394985-71e0c345-fdfb-45ee-a161-b72e4bb7492c.png#align=left&display=inline&height=210&margin=%5Bobject%20Object%5D&originHeight=210&originWidth=602&size=0&status=done&style=none&width=602)
- JVM规范没有明确规定初始化之前的四个阶段何时开始，但明确规定了初始化操作必须当且仅当在如下5种情况下进行：
   - 遇到new、getstatic、putstatic、invokestatic时
   - 对类进行反射调用时（java.lang.reflect）
   - 初始化一个类时，若其父类尚未初始化，则先初始化其父类
   - main方法所在的主类
   - [?类似于反射，但不是很理解，此处先照抄“深入理解JVM”]当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。
- 类加载过程
   - 加载
      - 流程
         1. 通过一个全限定名获取定义此类的二进制字节流
         1. 将字节流中的静态存储结构转化为方法区的运行时数据结构
         1. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口
      - 数组类，一种特殊的类
数组会被JVM生成为一层内部类型的包装类，该类型包含length属性和clone方法，可见性与其内部类型一致
   - 验证
对于已经在环境中反复使用的类，可以通过关闭验证缩短类加载的时间
      - 文件格式检验
第一阶段验证发生在字节流流状态下，通过验证后该类就会加载至方法区中进行存储，之后的验证便不再操作字节流
         - 检验文件开头的前四个字节是否为0xCAFEBABE
         - 检验class文件版本号是否支持
         - …
      - 元数据验证
该阶段主要检验类中信息是否符合java语义，例如是否有父类、是否继承了final声明的类、是否实现了接口中的所有方法等等
      - 字节码验证
验证方法的字节码是否会对JVM产生危害，例如是否程序会跳出当前方法所允许的范围等等
      - 符号引用验证
验证类中的符号引用是否可以转化为直接引用，例如所引用的类是否存在、所引用的类是否可以被访问到等等
   - 准备
为类变量分配内存并设置类变量初始值
      - 此处只对类变量（static）进行操作，不考虑实例变量，分配的内存也是方法区的内存。
      - 初始值指的是JVM中的默认值（0、false、null），而不是代码中指定的那个值。例如 public static a =				123会在准备阶段后变为0而非123，赋值为123的操作需要在初始化阶段调用`<clinit>()`方法
   - 解析
      - 解析阶段是JVM将常量池中的符号引用转化为直接引用的过程
      - 直接引用是可以直接指向目标的指针、相对偏移量或者能间接定位到目标的句柄
      - 类或接口解析，设当前代码所处的类为D，将未解析的符号引用N解析为类或接口C的直接引用
         4. 如果C不是数组类型，那么将其全限定名N传递给D的类加载器去加载类C，如果加载类C出现异常，则宣告失败
         4. 如果C是数组类型，则根据规则1加载数组内部的元素类型，随后由JVM生成对应维度的数组类型
         4. 如果上述过程没有出现问题，那么C已经称为虚拟机中的有效类或接口了，但在最后一步仍然需要符号引用验证，确认D对C的访问权限，如果不具备访问权限，则
      - 字段（Field）解析
         7. 如果当前类本身包含该字段，则返回这个字段对应的引用，查找结束
         7. 否则，查看接口和父接口，如果查找到，则返回对应的引用，查找结束
         7. 否则，从下往上递归查看父类，如果查找到，则返回对应的引用，查找结束
         7. 否则，宣告查找失败，抛出NoSuchFiledError异常
      - 类方法（static method）解析，设当前类为C
         11. 若C为接口，则抛出异常
         11. 在C中查找，如果查找到则返回引用，查找结束
         11. 否则，递归查找父类，如果查找到则返回引用，查找结束
         11. 否则，依次查找接口，如果查找到，则说明C为抽象类，返回AbstractMethodError异常
         11. 否则，宣告查找失败
         - 如果是查找到，进行访问限制的验证，如果不具有访问权限，则抛出IllegalAccessError异常
      - 接口C的方法解析
         - 在C中查找，若找到则返回引用，查找结束
         - 否则，递归查找父接口，若找到则返回引用，查找结束
         - 否则，宣告查找失败
   - 初始化
这个阶段简而言之就是JVM执行生成的`<clinit>`方法
`<clinit>`方法是编译器收集类中的类变量赋值和静态语句块中的语句合并生成的，JVM会保证子类的`<clinit>`执行前父类的`<clinit>`已经执行完毕
- 类加载器
   - “通过一个类的全限定名获取此类的二进制字节流”的模块称为类加载器
   - 被不同类加载器加载的同一个类会被当做不同的类，包括equals、instanceof等
   - 双亲委派模型
      - JVM中存在以下两种不同的加载器
         - 启动类加载器（Bootstrap ClassLoader）
            - C++实现的JVM的一部分
            - 加载`<JAVA_HOME>\lib`下的类，但具体的类由JVM指定，自定义的类即使放到该目录下也不会被加载
            - 启动类加载器不可以被java程序直接访问，如果需要将加载请求委托给启动类加载器，直接在相关方法使用null代替即可
         - 其他类的类加载器又可以划分为如下两种，由Java实现，独立于虚拟机，继承自抽象类java.lang.ClassLoader
            - 扩展类加载器
加载`<JAVA_HOME>\lib\ext\`下的类库，开发者可以直接调用
            - 应用程序类加载器
负责加载用户路径（ClassPath）上所制定的类库，一般情况下这是程序中默认的类加载器
      - 下图称为双亲委派模型
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100395055-f81dc409-0eb7-45e9-b79b-300d9f322709.png#align=left&display=inline&height=390&margin=%5Bobject%20Object%5D&originHeight=390&originWidth=356&size=0&status=done&style=none&width=356)
         - 双亲委派意思就是说，除了启动类加载器，其他加载器首先会将自己的加载请求委托给父加载器，如果父加载器反馈自己无法完成这个加载请求，那么才会尝试自己加载。
         - 这些加载器间的关系使用组合实现，而非继承
         - 双亲委派的意义在于，所有的类最终可能都由启动类加载器加载，这很大程度上避免了我们之前提到的“不同的类加载器加载的同一个类会被当做不同的类”的问题，这对维持java程序正确运行有很大的意义。这个模型不是强制要求的，但是是java设计者推荐给开发者的一种类加载器实现方式
      - 破坏双亲委派模型
         - JNDI
         - OSGi



## 虚拟机字节码执行引擎


- 栈帧
   - 方法运行产生的栈帧所需的空间都是在编译期就确定的，这个大小已经写在了类文件的每一个Code属性表中。
   - 典型的栈帧结构如下图，每一部分的解释间图后段落
![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100394991-a413002f-35f7-4889-b69b-05ae39f75b8c.png#align=left&display=inline&height=520&margin=%5Bobject%20Object%5D&originHeight=520&originWidth=484&size=0&status=done&style=none&width=484)
   - 局部变量表
      - 以slot作为存储单位，slot由0开始索引
      - 如果执行的为实例方法，那么0号slot存储的是调用者对象的引用，即this；随后是参数列表；再然后根据方法体内部定义的变量顺序和作用域分配其余slot
   - 操作数栈
方法运行产生的操作数栈也已经写在了Code属性表中
   - 返回地址
如果方法正常退出，返回地址由此处确定；如果方法出现异常未能正确返回，返回地址通常由异常处理表给出
- 方法调用
   - 名词：多态结果唯一的方法
类方法、`<init>`方法、私有方法、父类方法、final方法这类在多态情况下也可以保持结果唯一性的方法，由于他们不可更改并且指向明确，称之为多态结果唯一的方法，也称之为非虚方法。这些方法的引用都会在类加载的解析阶段进行解析出来。
   - JVM提供了5中方法调用指令，分别为
      - invokestatic
调用类方法（静态方法）
      - invokespecial
调用`<init>`方法、私有方法和父类方法
      - invokevirtual
调用虚方法，此处虚方法指的是多态结果可能不唯一的方法。
      - invokeinterface
调用接口方法
      - invokedynamic
略，Java语言尚未使用到这个指令，这是为动态解析调用准备的
   - 分派
      - 静态分派、重载解析（针对重载）
下面的代码会输出human，而非输出h对象所真实对应的类型。我们将Human称为h的静态类型（static type）或者外观类型（apparent type），将Man称为h的实际类型（actual type），对于同一个变量，静态类型和实际类型都是可以变化的，但静态类型最终是编译期可知的，而此处的重载又依赖于编译期选择恰当的函数（因为编译器要在此处确定**invokestatic**指令后的方法名参数），所以编译器依据静态类型，选择使用最为“适当”的方法，即Human作为参数的版本。调用print方法的字节码是这样的`invokestatic #4 // Method print:(Lcom/JVM/dispatch/Human;)V`，其中双斜杠注释指的是#4所对应的常量池中的符号引用字符串
```java
public class Dispatch {
public static void main(String[] args){
	Human h = new Man();
	print(h);
}

public static void print(Human h){
	System.out.println("human");
}

public static void print(Man man){
	System.out.println("man");
}

public static void print(Women w){
	System.out.println("women");
}
}

class Human{}
class Man extends Human{}
class Women extends Human{}
```

      - 动态分派（针对重写）
下面的代码将会输出man而非human，这是因为编译器无需选择哪个函数，只需要将调用print方法的对象压栈，然后**invokevirtual** print，具体解析重载方法的过程将有JVM递归搜索完成。需要注意的是，即使运行结果是根据实际类型来判定的，但是代码在进行静态分派时仍然使用的是Human.print方法，只不过JVM在运行时只取简单名（print）进行解析，详情见下文
```java
public class DynamicDispatch {
	public static void main(String[] args){
		Human h = new Man();
		h.print();
	}
}
class Human{
	public void print(){
		System.out.println("human");
	}
}
class Man extends Human{
	public void print(){
		System.out.println("man");
	}
}
class Women extends Human{
	public void print(){
		System.out.println("women");
	}
}
```

      - invokevirtual指令的运行时解析过程
         - 找到栈顶元素所指对象的实际类型，记为C
         - 如果类型C找到与参数常量中**简单名**和描述符都相同的方法，则进行访问权限校验，成功则返回引用；失败则抛出访问权限异常
         - 否则，按照继承关系从下到上依次搜索C的父类进行第二步的搜索和验证过程
         - 否则，抛出AbstractMethodError异常
      - JVM一般不会每次都进行上述解析过程，而是准备一张虚方法表，直接存储了每个方法确切的方法入口，以减少每次都要查找方法入口的开销
   - 单分派与多分派
- 静态语言与动态语言的区别：有类型的是变量还是变量值
- [?]invokedynamic



# JAVA内存模型与线程


## Java内存模型（Java Memory Model）


Java内存模型主要是定义程序中线程共享变量的访问安全，即虚拟机如何存取变量这样的底层细节


Java中规定所有对象都存储在主内存中，每个线程还有自己的工作内存（实际上是CPU Cache），线程工作内存中存储了自己所需的主内存中对象的副本，所有操作只能在线程自己的工作内存中进行，而不能操作主内存。它们间的关系如下图，模型中的缓冲是示意未赋给变量的值暂存的状态，实际上不存在一块物理缓冲区域：


![](https://cdn.nlark.com/yuque/0/2019/png/657413/1577100395012-08ace852-0a41-46d9-9fbd-1896e52448a2.png#align=left&display=inline&height=212&margin=%5Bobject%20Object%5D&originHeight=212&originWidth=764&size=0&status=done&style=none&width=764)


- Java内存模型定义了如下8中操作，虚拟机实现需要保证这8个操作都是原子操作：
   - lock：作用于主内存变量，设置某一对象为线程独占
   - unlock：作用于主内存变量，释放某一被锁定的对象
   - read：作用于主内存变量，把变量的值从主内存传输到工作内存中
   - load：作用于工作内存变量，从read操作得到的变量值加载至JVM栈中
   - use：作用于工作内存变量，它把工作内存的一个变量的值传递给执行引擎。JVM遇到使用变量值得字节码时执行该操作
   - assign：作用于工作内存变量，它把从执行引擎获取到的值赋给工作内存中的变量。JVM遇到给变量赋值的字节码时执行该操作
   - store：作用于工作内存变量，它把JVM栈上的值放回工作内存中，以便后续write操作使用
   - write：作用于主内存变量，它把工作内存中的到的变量的值放入主内存变量中
- 一些关于原子操作的规定：
   - read/load和store/write必须成对出现，但中间可以插入其他操作
   - 出现assign后工作线程工作线程一定要将其写回主内存，而不能选择抛弃
   - 不允许线程在无assign的状况下回写工作内存中的值到主内存中
   - 新变量只能在主内存中产生
   - 一个变量同时只能有一个线程对其进行lock操作，且lock操作是可重入的，即可以多次lock，但是必须进行同等数量的unlock才能解锁
   - **对一个变量进行lock操作，会清空工作内存中该变量的值，执行引擎使用时需要重新assign或load**
   - 不允许线程unlock一个不是该线程锁定的变量
   - **对一个变量进行unlock前，必须保证变量已经同步回主内存**
- happens-before:
   - 程序顺序规则：单线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作；
   - 管程锁定规则：一个unlock操作先行发生于对同一个锁的lock操作；
   - volatile变量规则：对一个Volatile变量的写操作先行发生于对这个变量的读操作；
   - 线程启动规则：Thread对象的start()方法先行发生于此线程的其他动作；
   - 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；
   - 线程终止规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行；
   - 对象终结规则：一个对象的初始化完成先行发生于它的finalize()方法的开始；
   - 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C；
- 对volatile变量的特殊规则
   - 关键字volatile是Java提供的最轻量级的同步机制
   - 两个语义：
      - 对所有线程是立即可见的
但这并不意味着基于volatile的变量是线程安全的，因为java中的运算并不是原子操作，如下例：
```java
volatile static cnt = 0;
//线程A
cnt = 1;
return -cnt;
//线程B
cnt = 2;
return cnt+1;
```

      - 
这部分代码再经过字节码重排以后可能会有如下执行顺序，仅为示意，并非真正的字节码：
```java
/*线程A*/
push 1
pop cnt		//线程A中的cnt=1

/*线程B*/
push 2
pop cnt		//线程B中的cnt=2

/*线程A*/
push cnt
neg		//使栈顶存储的值为-cnt,此时栈顶为-2而非期望的-1，同步错误
```

      - 
这样的话，即使volatile保证了立即可见性，也未能保证volatile变量的运算是线程安全的
      - volatile变量参与的指令不会被重排序
在JVM层面，内存屏障禁止指令重排序就是一项happens-before原则的具体体现，对volatile变量的写操作一定在读操作之前。
在汇编层面，会生成带内存屏障的汇编指令，禁止 CPU 对 volatile 变量的重排序。在X86里，内存屏障是使用 lock 修饰的指令，该修饰会使得CPU乱序发射时不会将后续指令重排序到lock指令之前，且当运行到该条指令时，当前内核会将CPU Cache写进内存，并无效化其他内核Cache中的此数据，实质上是一种 StoreLoad 屏障。
   - 所以在使用volatile时仍需注意基本原则：
      - 运算结果不应该依赖于volatile当前值，或者只有单一线程可以改变volatile变量的值
      - volatile变量不需要与其他变量共同参与不变约束
   - 典型场景
      - 状态检查：只有一个线程对volatile变量修写入权限，其他变量只有读取权限。
      - 一次性安全发布：双重检查锁定，即在单例模式中的应用
   - 对long、double变量的特殊规则
JVM规范允许虚拟机将未被volatile修饰的64位的数据拆分为两次32位操作，因而long、double的读写操作不具有原子性。但商用虚拟机都为64位数据实现了原子操作。
   - Java中的先行发生关系



## Java与线程，线程安全实现方法


- 阻塞同步（悲观锁）
常规的Synchronized和ReentrantLock
- 非阻塞同步（乐观锁），需要硬件支持
   - 先操作 ，再检测是否冲突，如果冲突再采取补救措施（通常为不停重试）
   - 比较并交换（Campare and Swap， CAS）
需要三个参数：内存地址、旧值、新值
当且仅当内存位置为旧值时，将其更新为新值，否则不执行更新。最终无论如何都会返回旧值。上述操作为一个原子操作
   - 无同步方案
将需要访问的数据限定在一个线程中，例如，使用阻塞队列一放一取，保证每个时刻只有一个线程在操作指定对象；使用ThreadLocal等等
- 锁优化技术
   - 自旋锁（上下文切换代价较大）
基于`互斥锁 -> 阻塞 –> 释放CPU`这个流程中线程上下文切换代价较大的思想，并且共享变量的锁定时间通常较短，此时可以让线程通过自旋等一会儿，称之为自旋锁。
   - 锁粗化(一个大锁优于若干小锁)：一系列连续操作对同一对象的反复频繁加锁/解锁会导致不必要的性能损耗，建议粗化锁
一般而言，同步范围越小越好，这样便于其他线程尽快拿到锁，但仍然存在特例。
   - 偏向锁(有锁但当前情形不存在竞争)：消除数据在无竞争情况下的同步原语，提高带有同步但无竞争的程序性能。
   - 锁消除(有锁但不存在竞争，锁多余)：JVM编译优化，将不存在数据竞争的锁消除
