# 个人感受


中台这本书本质上来讲并不是一本技术书，而是在探讨业务理解，对技术的讲解只是对涉及到的点稍微展开讲讲阿里的一些实践。中台是业务、技术、组织架构的综合产物，业务为核心，技术是手段，组织架构作辅助，在更高层面封装为可以对外提供服务的开放服务化平台。从技术角度来读这本书收获不是很多，因为很多东西在行业内已经普遍应用了，但是能感觉到如果从业务角度去看（我还没到那个 level），这本书还是包含了很多很有价值的观点的，有重读的价值。


下图给一个阿里中台和其“厚平台，薄应用”的架构图，其中共享业务事业部便是中台所在：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581176464060-d30dd078-550b-48ab-ab9d-7f49943f406b.png#align=left&display=inline&height=460&margin=%5Bobject%20Object%5D&name=image.png&originHeight=613&originWidth=815&size=230500&status=done&style=shadow&width=611)


我觉得可以从如下两个方面去讨论中台的构建：


**技术层面**


- 服务框架
- 服务发现
- 数据库分库分表
- 异步化
- 缓存
- 分布式日志服务
- 限流与降级
- 全链路压力测试
- 业务审计平台
- DevOps



**服务化层面**


- 服务治理
- 开放平台
- 服务生态闭环构建



读这本书的时候试图用思维导图去记笔记，但是发现思维导图并不适用于细节的记录。但是思维导图再转成笔记太消耗时间了，所以下面能用图表现的地方就直接贴图了。并且反思一下，其实很多细节记录没必要，以后要改改记笔记的毛病了。


# 第四章 共享服务中心建设

---



![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581151985274-c6f2da91-7758-48c6-9751-e9f718524aab.png#align=left&display=inline&height=311&margin=%5Bobject%20Object%5D&name=image.png&originHeight=470&originWidth=1129&size=67396&status=done&style=shadow&width=746)


# 第五章 数据拆分实现数据库能力线性扩展

---



![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581152153910-18722c6d-a894-40a2-811c-f5cfc2654b89.png#align=left&display=inline&height=350&margin=%5Bobject%20Object%5D&name=image.png&originHeight=631&originWidth=1345&size=86323&status=done&style=shadow&width=746)


# 第六章 异步化与缓存原则

---

## 异步化


本章的核心思想都在于异步化，具体的落地手段是通过消息队列。异步化可以带来两个主要好处：


1. 不同服务子链路间业务逻辑间并行执行，大幅提升效率；
1. 避免了大型事务执行过程中产生的大量的锁，降低了锁冲突的概率。



## 分布式事务


开始简要介绍了 CAP、BASE 和具体的一致性协议 Paxos、Raft，以及具体的 2PC 分布式事务实现方法，这些内容不再赘述。


这一节主要内容是柔性事务（满足 BASE）的一些思路与实践。


柔性事务的思路有以下三点，也使注意要点：


1. 引入日志与补偿机制，通过正向补偿（重试）或反向补偿（回滚）推动事务完成；
1. 可靠消息传递保证消息至少传第一次，但可能传递多次，因此开发者需要注意服务的幂等性；
1. 使用无锁方法避免大量锁冲突。



## 阿里的分布式事务实现方案


### 消息式分布式事务


思想如图，具体解释见书中该图解释：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581174488595-6773d5d3-2827-4330-aaf4-ed978cc99b69.png#align=left&display=inline&height=192&margin=%5Bobject%20Object%5D&name=image.png&originHeight=256&originWidth=814&size=93361&status=done&style=shadow&width=611)


简单来说就是每一个服务节点事务执行成功后向消息队列上推送消息通知后续服务节点开始执行，直至所有参与节点事务都执行完毕。需要注意的地方在于，消息式分布式事务使用正向补偿机制，不会通过回滚异常事务产生的修改。


举个例子，下图是淘宝下单流程，涉及到了订单服务、支付服务等一系列服务：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581174683178-249a92a7-ce90-4fb5-a122-9861c55667cf.png#align=left&display=inline&height=206&margin=%5Bobject%20Object%5D&name=image.png&originHeight=411&originWidth=814&size=150344&status=done&style=shadow&width=407)


如果出现了异常（库存不足或支付超时），则会出现以下的事务正向补偿，不使用回滚，而是发送补偿的“撤回”、“退款”消息继续推进事务向前进行：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581174761826-34ddb133-3261-464c-874e-a9a031bf3093.png#align=left&display=inline&height=200&margin=%5Bobject%20Object%5D&name=image.png&originHeight=400&originWidth=817&size=114599&status=done&style=shadow&width=409)


### 支付宝 XTS （基于 TCC 思想）


TCC 分别为：


- Try：完成业务检查，预留业务资源
- Confirm：真正完成业务，只是用 Try 预留的资源，满足幂等性
- Cancel：取消执行业务，释放 Try 预留的资源，满足幂等性



通过实现框架的 Try / Confirm / Cancel 接口，实现了在业务逻辑中完成事务的操作，避免大规模的**数据库锁**（2PC 同步阻塞的主要问题）。由此引发的问题便是强依赖于开发者正确的编程。个人感觉这个方法很坑爹。


### AliWare TXC


一个用于解决 XTS 问题的框架，有以下几个特点：


- 基于 2PC 思想
- 标准模式下不需要开发者手动编写回滚逻辑
- 默认隔离性是最低的读未提交，会出现脏读的可能，但支持 select for update



其架构如下图所示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581174998887-d367e148-afce-4230-96b4-672133b32a6c.png#align=left&display=inline&height=249&margin=%5Bobject%20Object%5D&name=image.png&originHeight=497&originWidth=810&size=154007&status=done&style=shadow&width=405)


简单来说，TXC Server 作为切面接管了业务逻辑到数据的访问，在这之中自动管理事务的执行顺序、数据回滚（undo log、redo log）。在事务执行过程中，TXC Server 直接记录数据库中数据的修改，然后在事务回滚时通过其记录的 undo log 覆写这些数据将其恢复至事务执行前的状态。这种架构将原本放在数据库中的锁提升到了 TXC Server（伸缩性好） 中，实现更为轻量且易于水平扩展。


TXC 事务执行过程如下图所示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581175247902-d7db6b00-6d7d-46b7-9b6b-5ac00be37e84.png#align=left&display=inline&height=378&margin=%5Bobject%20Object%5D&name=image.png&originHeight=756&originWidth=801&size=162513&status=done&style=shadow&width=401)


1. 应用向 TXC Server 发起事务，TXC Server 根据收到的 SQL insert / update /delete 语句反向生成查询语句，提前获取到对应数据，保存为 undo 日志用于事务回滚；
1. 应用执行事务的 SQL 语句完毕后，TXC Server 再次查询获得修改后的数据，并保存为 redo 日志，用于业务回滚前的校验；
1. 当事务成功提交后，删除 undo / redo 日志；如果需要回滚，则根据上述 undo 日志生成 undo SQL 回滚数据库。



TXC Server 有一个小问题：上述回滚过程中，TXC Server 会先通过 redo 日志中的数据与数据库中的实时数据做对比，如果有非 TXC 管理的应用修改了数据，则这两个值可能出现不一致情况，此时需要告警，引入人工干预。


# 第七章 数字化运营能力

---

 所谓数字化运行服务，指的是如何在分布式的情况下快速排查系统错误。在分布式的环境下，一个调用链路涉及的数百个服务部署在数百个机器上，如何确定某一次错误请求经过的所有服务节点，并找出其对应的错误日志呢？由此本章引出了分布式日志服务。


分布式日志服务是一个集中式日志收集服务，例如文中提到的阿里鹰眼、Zipkin，他们都是基于 Google Dagger 的思路涉及的。其中，阿里鹰眼的架构如下图所示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581172511846-a580be2f-b42c-40ea-859e-1e020b08c6f7.png#align=left&display=inline&height=388&margin=%5Bobject%20Object%5D&name=image.png&originHeight=517&originWidth=811&size=208712&status=done&style=shadow&width=608)


简单来说，鹰眼通过部署在虚拟机中的 agent 读取业务日志并上报给鹰眼服务端，鹰眼服务端对日志进行处理与存储，并通过在线流式处理或离线分析提供数据分析的能力。一次请求链路（一般是多次 RPC / HTTP 调用）通过一个特定 TraceId 标识，为了方便调试，日志应包括如下内容：TraceID、RPCID、开始时间、调用类型、源ip、目的ip、处理耗时、ResultCode、数据大小。


一次链路追踪的在分布式日志服务上可视化效果如下图所示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581172765370-a411f9bd-a15b-4570-a4dd-640461175451.png#align=left&display=inline&height=408&margin=%5Bobject%20Object%5D&name=image.png&originHeight=543&originWidth=814&size=328930&status=done&style=shadow&width=611)


# 第八章 打造稳定性能力

---

## 限流和降级


通常情况下，使用硬件性能指标和 QPS 作为限流的指标。平台要具备限流能力，首先需要对服务器的服务能力有准确评估，知道服务实例的部署量到底最大能满足多少业务的处理要求。这需要通过压力测试的实现。


实现限流有两个实施点，分别为**接入层**和**服务层**。


### 接入层限流


下图是通过接入层的 Nginx 服务器实现限流的示意图：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581090360646-dcd832e5-fb89-433c-9dba-2563de365046.png#align=left&display=inline&height=338&margin=%5Bobject%20Object%5D&name=image.png&originHeight=676&originWidth=769&size=157083&status=done&style=shadow&width=385)


淘宝通过 Nginx 扩展组件 TMD（Taobao Missile Defense） 实现了接入层限流的主要工作，包括域名类限流、Cookie 限流、黑名单策略等等。当目标服务器内存、CPU 利用率达到一定的阈值后，会执行相应的动作，比如直接返回 503 或 504 等，直至服务器恢复。


除了限流功能外，TMD 还提供了配置界面，方便开发人员在统一平台上直接配置限流策略。此外，在被限流的情况下，用户会被引导至和淘宝风格一致的限流页面。


### 服务层限流


阿里在服务层使用 Sentinel 实现限流，这是基于 AOP 思想的服务器层限流实现。Sentinel 实现了授权、限流、降级、调用统计四大功能模块：


- 授权：通过黑白名单对 HSF 接口和方法调用权限进行控制；
- 限流：对特定资源的调用进行保护，防止资源过度使用，使用的算法为 Leaky Bucket；
- 降级：判断依赖的资源的相应情况，当依赖的资源响应时间过长时进行自动降级，并在一定时间后自动恢复；
- 监控：提供全面运行状态监控，实时监控资源调用情况（QPS、响应时间、限流降级信息等）



Sentinel 整体结构如下图：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581092343345-c9f5eddd-368c-487d-8cfb-1809249c9251.png#align=left&display=inline&height=249&margin=%5Bobject%20Object%5D&name=image.png&originHeight=498&originWidth=762&size=124022&status=done&style=shadow&width=381)


## 流量调度


在微服务的背景下，完成一项服务需要很多服务的参与，若其中某一台服务器出现了问题，即单点故障，便会影响整个链路的功能。导致单点故障的一些原因如下：


- JVM 假死
- JVM GC 导致无响应
- 部分应用启动时导致的负载飙高
- CPU 核数超配（超配指的是一个物理核心对应了大于一个虚拟核心，适当超配有利于利用长尾应用用不到的闲置资源）



为了避免这种单点故障影响服务，我们需要实时地监控服务，动态分配服务流量，避免单点故障导致服务不稳定。


简单来说，阿里实现了一个流量调度平台，定时（书中说 5 秒）访问服务器、容器（Tomcat）、应用以获取服务器和服务的指标，例如 CPU、Load、QPS、Tomcat 线程池等。通过采集这些指标，流量调度平台可以调整 HSF 中 RPC 的权重，使服务器都维持在可以提供稳定服务的负载范围内。其整体结构如下图所示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581096717156-39882a86-1c09-4536-8cc5-cc025b7b0d6a.png#align=left&display=inline&height=258&margin=%5Bobject%20Object%5D&name=image.png&originHeight=515&originWidth=797&size=146366&status=done&style=shadow&width=399)


## 业务开关


没什么好说，线上配置平台，通过推送配置的方法达到不重新上线便能在一定程度上控制服务业务逻辑。


## 容量压测及评估规划


压测的主要分为两个部分：**容量压测（方法）**和**容量分析预测（目的）**。


书中介绍一种容量压测方法：将生产环境请求通过调整负载均衡权重（RPC）的方式引流到单一服务器上，并设置某些指标（CPU、MEM、IO）的上限阈值，自动控制压测的结束时间。压测结束时的 QPS 便是单机可提供的最大 QPS。这种方式通过完全真实的生产环境数据进行压测，比单纯的生成数据得到的结果更加真实、可靠。


### 全链路压测平台


所谓全链路压测指的就是在生产环境中对服务链路进行全覆盖的压力测试。为什么要在生产环境？我认为主要原因是在测试环境搭建完整的服务是不显示的。


阿里通过对请求设置不同的标志位来区分业务流量和压测流量，并对应用、中间件、数据库、安全等均做了改造以支持全链路压测。通过对海量参数的组合构建压测请求，进行数小时的压测以验证所有可能服务链路的稳定性。


## 业务一致性平台


业务一致性平台用于实时检测业务数据的不合逻辑的情况，例如优惠前显示已使用但未减免费用，这种情况多数是由于开发者逻辑错误导致的。业务一致性平台会实时监测数据一致性，如果发现不合逻辑的数据便会发出告警。平台使用规则引擎对数据进行验证分析数据的正确性，开发者可以在相关平台编写自定义的业务数据规则。该过程如下图所示：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581097493507-4a84b326-2d31-4462-aeb4-b61cb63aa99a.png#align=left&display=inline&height=269&margin=%5Bobject%20Object%5D&name=image.png&originHeight=359&originWidth=800&size=93278&status=done&style=shadow&width=600)


# 第九章 共享服务中心对内和对外的协作共享

---

这一章讲的是阿里如何将已有服务转化为服务平台上可以上架的服务，并开放给企业内（其他团队）和企业外（例如为淘宝店主开发 ERP 系统的开发者）的开发者的。核心思想在于服务平台的构建，并且通过服务平台，构建生态圈。


首先作者明确了两个概念，服务和服务化：


- 服务：这是一个名词。我们通常说的服务指的是服务端暴露出来的某个接口，如果为这个接口附加上策略、限制与描述，便成为了 SOA 中的组件化服务；
- 服务化：这是一个动词，更像是商业策略，核心是从产品能力转化为服务客户的能力。服务化的思路就是把产品和服务的中心转化到用户身上，以方便用户使用、降低用户成本为目标。如果用户是开发者，就从开发需求出发（SDK）；如果用户是运营，就从运营角度出发（运营平台、数据分析）。



如何从“服务”转化为服务化后“可以上架的服务”呢？需要对既有裸服务的四步改造：


1. 确定服务对象，是 API 还是 SDK 还是解决方案？
1. 对服务对象进行封装，使其具有规范化描述，可以受服务治理平台的治理，此时 API 便成为了组件化服务；
1. 服务化实施，API as Service? Product as Service? Solution as Service? 这是一个逐步升级的过程。



服务平台有三种角色，分别是服务提供者、服务消费者和平台自身。其各自分工见下图：


![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1581173717958-d1e023b2-671a-4b07-a2fd-d2e4e14f8664.png#align=left&display=inline&height=542&margin=%5Bobject%20Object%5D&name=image.png&originHeight=722&originWidth=814&size=307717&status=done&style=shadow&width=611)
