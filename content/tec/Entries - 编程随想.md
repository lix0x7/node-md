# 学习时候应当思考的东西
学习知识点A应当思考如下问题：

1. A是为了解决什么问题而存在的？Why
1. 现有的问题的严重性，什么时候会成为问题？
1. A是如何解决问题的？How
1. A在何种程度上解决了问题？
1. A解决问题引入了什么新的问题？
1. 该在什么时候引入A，而什么时候维持旧有做法？When（实际上同2，是在学习完成后的一种复习机制）



学习完后及时输出一份小总结，可以忽略细节，但主干内容要记清楚。此处利用费曼学习法的思想，要尽快输出。


# 复用
复用可复用的部分，但不要过度复用而忽略了扩展性。对于 MVC 架构，C 与 C 之间如果需要复用，是因为他们能共用参数处理（格式检查、转换）、返回包装，也就是说，他们在接口形式上是一致的，区别只有接口名；Service 与 Service 之间如果需要复用，是因为他们对于某些逻辑、数据的处理是一致或者可以复用的。能不能复用，如何复用取决与模块与模块之间的关系（平级 or 上下级）。

# 切面编程、命令式编程
一般而言，很少有需要进行切面编程的。大多数的切面编程都是权限验证、错误处理、接口级别的缓存这种十分通用的操作，不要什么都尝试用切面编程解决，该抽象为方法的就老老实实抽象。尤其是使用 Express.js 这类切面编程友好的框架，就会不自觉地把切面编程当命令编程使用，最后代码结构混乱、不好维护。

# 依赖注入思想万分重要，是解耦的核心
对于类成员，不直接在类内进行实例化，而是通过初始化方法或 set 方法单独传入，这样可以随意组合组件，根据不同的场景使用不同的组件，大幅度解耦。在 SpringMVC 中，我们把注入的工作交给 Spring 容器，更进一步降低了开发工作量。

在web服务开发过程中，最重要的其实就是根据运行环境的不同注入不同的组件，例如单元测试时的logger为console，但是线上运行需要变更为FileLogger。

# 异常处理
使用统一的异常处理，切面实现，并作为兜底，保证不外泄异常到外部。

# 配置与硬编码
不要过于追求“万物皆可配置”，配置会带来额外的工作量。不易发生变动、数据量不大的部分，该硬编码就硬编码，追求可配置反而会变得更麻烦。

# 使用异步而非同步函数
console.log 就是一个典型的同步函数，同步 IO 输出是极大地影响性能的点。日志库的实现一般都是维护内存队列，定时刷盘。

# 在某些时候，约定大于配置，保持自己的编程习惯
能通过约定和习惯解决的问题不要首先依赖于配置。例如，无论什么地方，都用毫秒而非秒，对于特定的终端场景再做转换。

之所以“在某些时候的限定”，是因为“约定大于配置”本身是隐晦的，约定只适用于自己绝对熟悉的某些领域，里如果设计模式、时间单位、工具类，而不适合合作类项目。Springboot 强调约定大于配置，但其很多约定初学者无从得知，由此产生很多困惑。

Python Zen 中的一条提到 `Explicit is better than implicit` 。所以说，绝大多数情况下，程序还是应直观简洁。

# 动态语言中的类
对于动态语言开发，该抽象为类的时候仍然用类，不要随意使用未定义的数据结构。随意定义的数据就够只适用于中间过程，而非抽象实体。


# 使用包装类而非原始类型
类成员变量的类型应始终使用包装类型而非原始类型，使用原生类而非原始类型。这样可以明确地区分无值（null）和 0 值。尤其是使用 mybatis 这类 ORM 工具进行动态更新时，可以明确知道哪些字段有值需要更新，而哪些字段没有值不需要更新。


# 表主键使用 `表名+id` 
例如，project 表使用 `project_id` 作为主键，而非直接使用 `id` ，这样更清晰。相关讨论见 [StackOverFlow - Is it better to name the primary key column id or *_id?](https://stackoverflow.com/questions/6469730/is-it-better-to-name-the-primary-key-column-id-or-id)


# 程序的本质
当然， `程序=数据结构+算法` ，但这个问题是从另外一个角度看问题，我认为 `程序=输入+处理+输出` ，尤其是对于web服务，也就是接收用户输入和数据库数据数据，处理后给用户输出。所以我们编程也应该从这三个基本元素去建模。通常情况下，数据库的读取和写入是在一起的，所以按照ddd的要求，我们将其都是现在repository里，而数据处理的部分使用独立的面向对象编程方法实现。如果数据的读取和写入在不同的位置（例如从MQ读入，写入到DB；从缓存/DB分级读入），也可以该逻辑实现在repository中。repository正如其名，就是一个数据仓库，我们的程序从里面拿出来东西，再把需要持久化的东西放进去存起来，至于如何实现repository，业务逻辑并不关心。


# 最好的语言是 Shell 和 C
在这个Linux作为基石的互联网时代，Shell和C是最好的编程语言，且均是Unix默认支持的。

Shell可以帮助我们实现很多自动化脚本，这些脚本在开发和生产环境都可以无缝的使用、可以无缝地调用Shell命令，可以极大降低繁复操作带来的效率损耗；后台开发的各种工作都需用到命令行，命令行便是最基本的Shell脚本，学会、用好Shell是当今时代作为开发的基本能力。虽然Python功能更强，在Shell胜在高度与系统融合且语法稳定，不会出现环境没有Python、没有Python3、不支持指定版本语法的各种奇奇怪怪的问题。

C语言则是Linux的开发语言，Linux的系统调用也通过C语言方法暴露出来。此处的学好C不单单是这门语言，更是背后一系列的Linux调用、运作原理。因为C是最贴近操作系统的高级语言，而其他语言也最终都只是通过C语言的系统调用实现自身的各种特性。所以，与其在各种高级语言的语法、库中间迷失，不如抓住其根本的C语言和系统接口，从底层了解系统运作。最典型的例子：学习Java的线程与IO、NIO，便不如先学习Pthreads和select/poll/epoll/io_uring。


# 数据流动方式（同步、异步、阻塞、非阻塞）
这点指的是整个硬件软件体系中，系统和系统间的数据如何发生交互。其实这种流动方式体现在高级语言层面，就是IO的方式：


## 同步、异步
同步异步指的是caller和callee间的**消息通信机制**，也就是如何在两者间传递callee的返回值

- 同步方式指的是caller调用callee成功后一定能直接拿到结果，拿不到则callee不会返回；
- 异步指的是A调用B后不会直接拿到结果，而是等待B通过回调、信号等方式的通知A执行结果


## 阻塞、非阻塞
阻塞非阻塞指的是caller**在等待调用结果（消息，返回值）时的状态**

- 阻塞指的是caller在等待调用结果时挂起，不能处理任何其他事务
- 非阻塞指的是caller等待调用结果时不会将自己挂起，立刻拿到返回数据（无论是否存在）后做进一步处理

需要注意的是，阻塞是操作系统级别的概念，指的是进程不再占用CPU时间片，对硬件并不适用。

同步非阻塞IO的情况下，caller和callee之间传的结果是「可否进行IO」，「同步」指的是返回结果会通过IO的系统调用直接返回，「非阻塞」指的是这个判断不会将当前进程挂起。


## 几个不同的层次看待数据流动的例子
### CPU
同步：

- X86 CPU从MEM获取数据 `movl 8(%ebp), %edx` 这一指令，是同步等待内存地址位置的数据进入edx寄存器。该指令完成后，数据一定在edx寄存器中，所以是「同步」的；对于CPU流水线来说，因为流水线不能在这期间处理其他事务，所以此时是「阻塞」的。
- CPU轮询（poll）网卡中的数据，检查是否有数据。如果网卡有数据，返回数据字节数；如果没有，返回0。这个过程中，CPU在一次询问结束后总能拿到这个字节数结果，所以是「同步」的。

异步：

- 中断：CPU接受网卡DMA发来的中断，执行Interrupt Handler获取数据；CPU接受键盘中断，获取键盘输入的数据。

### 高级语言编程
同步阻塞：unix中使用read从socket读取数据，返回值便是读取到字节数，这说明该操作是「同步」的；如果没有数据可供读取，当前进程挂起，这说明该操作是「阻塞」的。
同步非阻塞：unix中使用read从O_NONBLOCK标志的socket读取数据，返回值是读取到的字节数，这说明该操作是「阻塞」的；如果没有数据可供读取，直接返回0，这个调用过程中进程不会挂起，这说明该调用是「非阻塞」的。
异步：Node.js中调用一个异步请求方法，获取结果是通过caller调用时传入回调函数实现的，并且此时caller和callee是并发执行的，所以该过程是「异步」的。


### 系统架构
同步阻塞：RPC的调用与等待返回结果
同步非阻塞：调用方创建任务后，轮询检查任务创建状态
异步：消息队列、回调接口


# 要形成自己的开发方法论，思维模板
形成自己的开发方法论，可以大幅度降低开发耗费的时间和精力。方法论应包括但不仅限于如下方面：


1. 代码
   1. 基础CRUD及其关联资源CRUD快速实现，基于DDD思想
   1. 形成自己的一套模板代码
      1. 开源工具库
      1. 封装工具类的思维（不一定要有模板，但要有基本的封装方法论）
   3. 变量命名规范，例如：
      1. 事件名
         1. onIngressCreatedEvent
      2. 类名
      2. 局部变量名
         1. __s：表示列表(list)值
         1. __Map
         1. __Req
         1. __Rsp
         1. __Buffer
         1. __Time 毫秒级时间戳
         1. __Total 总量
         1. __AccuTotal 累积量
         1. __Avg 平均值
      4. 常量名
         1. CONST_VALUE_LIST
      5. 数据库字段
         1. createdBy
         1. updatedBy
         1. createdTime - 毫秒级时间戳
         1. updatedTime - 毫秒级时间戳
         1. __Date - 整型日期，例如 `20200101` 
2. RESTful API 规范与习惯
2. Project Layout / Readme Template
2. 微服务构建
   1. 基于DDD的领域分割方法（核心子域、通用子域、支撑子域）：一个同样的单词出现在了同一个场景下，但是表达了两个含义，应当考虑拆分为两个子域
   1. 聚合划分：理想情况下，一次事务只更新一个聚合根。实际编码中，可以接受3个以内的聚合根更新
   1. 事件机制
5. 架构设计思路
   1. 熟悉常见的系统方案实现，例如用户系统、权限系统、IM系统等



# 做一个功能的顺序与思考

1. 从产品维度问问题：这个功能的使用对象、操作流程、期望结果，这是不是真实的需求？真实的需求是什么？
1. 识别实体与实体间关系：这个功能对应的操作对象是什么？对应DDD的聚合根是什么，对应的实体是什么？实体间关系是怎样的？
1. 对操作对象的操作（e.g. CRUD）都有哪些？
1. 这个功能是否有副作用？是否对其他系统有依赖？
1. 决定开发采用 DDD 还是 事务脚本，一般情况下是这种方案，不要太死板
1. 根据 RESTful API 规范设计接口，结构上与 DDD 维持一致
1. 根据 DDD 规范编写业务逻辑
1. 编写单元测试



# 做一个系统架构设计的顺序与思考
参考：[https://github.com/donnemartin/system-design-primer#how-to-approach-a-system-design-interview-question](https://github.com/donnemartin/system-design-primer#how-to-approach-a-system-design-interview-question)


1. 描述用户使用场景、限制条件和前提假设
   1. 目标用户
   1. 使用方式
   1. 各种量级：用户量级（C10K...）、数据量（存储系统选型）、QPS
   1. 系统输入、输出与操作分别是什么
   1. 读写比例：重写轻读系统or重读轻写？
   1. 实时性要求？
2. 简单画一下系统架构图与关键组件，描绘数据流向
2. 设计并编码技术细节
2. 优化与Scalability：缓存、数据库分片、读写分离、负载均衡、水平扩容等

# 后台项目目录

- doc: API文档、依赖的外部资源文档
    - api.doc
    - 开发业务资料
    - 上线相关：每次上线需要的sql文件、todo等
- build: 构建、编译脚本
- src: 源码
- readme.md
- .gitignore

# 后台项目 README 目录

参考：[RichardLitt/standard-readme](https://github.com/RichardLitt/standard-readme)


- 背景 - Background
   - 业务背景
   - 遇到的问题
   - 为什么建立此项目
- 构建方法 - Build
- 启动脚本、命令 - Start
- 访问使用 - Usage
- 相关资源链接
   - 部署地址
   - 服务发现
   - 流水线
   - 监控、告警
- 部署方案 - Deployment
- 网络拓扑 - Network
- 开发需知 - Development
   - 依赖组件
   - 配置项
- 维护人员名单 - Maintainer
- Contributing
- License


# 业务接口文档 API Doc 目录

- 服务简介
- 接口信息
   - 服务地址
   - 通用请求信息
      - `content-type` : `application/json` 
      - sample
   - 通用返回信息
      - sample
   - 业务错误码
   - 实体
   - API
      - 获取列表API
         - path: `POST` `/sample/list` 
         - 请求参数
            - pageSize:   `Integer` 参数 1 的说明
         - 返回结构
            - status
            - msg
            - data
               - list[]: `string[]` 列表
                  - data: `string` 列表项信息


# 如何计算排期？
计算排期要区分不同的功能类型，对于不同的功能有不同的实现时间要求，将其相加后再留适度的 buffer 即可。越大的项目，排期越不准，应当向产品明确出来这种风险。如下是几种常见情况，仅供参考：


- 无现有数据结构情况下，每四个领域模型与库表设计，1天
- 新项目环境搭建，1天
- DB-based CRUD 开发
   - 增删改查接口开发，1天
   - 测试，1天
- 外部稳定系统对接
   - 熟悉系统，1天
   - 每2个接口，1天
   - 涉及到外部人员的联调，每2个接口1天（时间不确定，有风险）
- 前后端联调，每4个基本接口，1天
- 联调和测试阶段要占用 0.5 的时间，虽然开发已经完成，但是需要排查问题、修改 bug



# 不要过早优化、确定瓶颈后再优化
代码在实现阶段可以考虑到初步优化（缓存、并发请求、异步），但不要具体实践，除非性能已经是明确的开发目标了。因为很可能开发完成后，一开始想到的优化点并不是系统实际运行真正的瓶颈。


优化代码需要明确瓶颈后再进行优化，避免盲目优化，吃力不讨好。


# 使用领域对象尽可能缩小上下文
大型项目编写业务逻辑困难，代码散乱的一大原因在于口口相传，一旦传授的“知识”有丢失，那么后来的程序员很容易另起炉灶，把已有的逻辑又实现了一遍。最典型的情况就是枚举量都堆在一个叫 `constant` 的文件里。实际上，枚举量是最和领域对象强相关的常量，他们应当和领域对象绑定在一起，从代码实现的角度来看，他们应当是领域对象类的静态成员常量。


传统的三层架构把业务员逻辑写在service，把常量写在统一的constant里，把模型schema写在model下，这就导致了一种现象：明明聚合的业务概念，被打散到三个不同的文件中。


缩小上下文后，我们将所有的枚举值、业务逻辑都挂在model上，这样围绕model构建统一的业务概念，将代码的上下文缩小到这个model对象上，开发人员需要的任何变量、常量都可以在一个地方找到，相比去翻多个文件来说，心智负担会极大的缩小。


# 复杂的领域对象应该是自解释的

领域对象应当包含一个元信息描述符，可以对每个成员字段进行解释，包括但不仅限于如下信息
- 字段
- 字段展示名，例如中文名
- 注释，包含对字段的解释即描述

在 Java 中，一个类的描述信息可以实现为一个全局静态常量 `<Domain>Descriptor`

上述描述符可以用于校验（`validate`）领域对象。

## 上下文零散的一个例子
我们想要编写一个project的CRUD，它有一些需要判断的逻辑，例如： `项目状态` 为 `正常` 的时候才可以修改，我们很容易写出这样的代码：
```javascript
// constant.js
const PROJECT_STATUS = {NORMAL: 0, BANNED: 1, DELETED: 2}	// 项目状态标记

// projectService.js
const deleteProject = function(projectId){
	// ... load project from db
  if (project.status === constant.PROJECT_STATUS.NORMAL){
  	project.status = PROJECT_STATUS.DELETED;
  }
  // .. save project to db;
}

// project.js
class Project{
	status;
}
```


上文实现方案有两个问题：


1. 文件散落在 constant.js / service.js / project.js 三个文件
1. project的内部状态 NORMAL 暴露到了 service 中，而实际的调用者不应感知到这个常量，而是直接获得“正常”与否这个布尔值的结果



上文代码修改以后可以将所有的量归于 project.test 一个文件，并将同样将业务逻辑收束：
```javascript
// project.js
class Project{
  status;

  static STATUS = {NORMAL: 0, BANNED: 1, DELETED: 2};	// 项目状态标记

  isNormal(){
    return this.status === Project.STATUS.NORMAL;
  }
}
```


其实这也就是 OO 所强调的封装，但是在事务脚本大行其道的时代，似乎都被抛弃了。最明显的地方在于，将 `NORMAL` 这个概念封装，如果未来我们需要扩展一个“试用”的状态，这个状态仍然是正常的，但属于另外一个状态了。我们只需要修改 `isNormal` 方法。但对于事务脚本，我们需要注意修改所有使用了类似的语义地方。

# 规范

## 命名规范
业务、项目、代码根据如下这个顺序来命名字段，降低思考负担。

- 环境：例如 prod / test
- 业务：例如 blog / shop
- 服务类型：例如 server / gw / fe，分别代表后端服务、网关（接入层）、前端服务
- 子域：例如 article / feed / user
- 聚合根 / 实体：例如 timeline
- 属性：例如 id / type / status / isDelete / createBy
- 数据库字段
    - 实例名：与业务名相同
    - 表名：与聚合根相同
    - 列名：与属性名相同

由此很容易定位一个对象所属的语境，例如 prod_blog_server.article.timeline.status
如果大语境下已经包含了其中某些字段，则**务必**要省略这些部分，避免冗余。

不同系统间的核心命名尽可能保持一致，例如包含了前后端服务的单体 Git 仓库名为 `blog` ，则前后端服务在 K8s workloads 命名分别为 `fe-blog` / `server-blog` ，后台服务日志主题命名为 `server-blog` ，这样可以避免不同系统间不一致的命名引发记忆负担。

与此保持一致的命名还应有开发阶段涉及到各个方面，例如浏览器收藏夹、密码管理工具文件夹、文件系统文件夹。

## 代码分段与排版规范

```java
public class MetaData {

    // -------------------------------------------------
    // static fields
    // -------------------------------------------------
    
    // -------------------------------------------------
    // instance fields
    // -------------------------------------------------

    // -------------------------------------------------
    // Enum / Inner Class 
    // -------------------------------------------------
    
    // -------------------------------------------------
    // public methods
    // -------------------------------------------------

    // -------------------------------------------------
    // protected / private methods
    // -------------------------------------------------
    
    // -------------------------------------------------
    // getters / setters
    // -------------------------------------------------
    
}
```

## Git commit 规范
主要参考 [angular - Commit Message Guidelines](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#commit) ，但精简如下，内容全小写：
```
<type>(<scope>): <short summary>
  │       │             │
  │       │             └─⫸ Summary in present tense
  │       │
  │       └─⫸ Commit Scope: 相关业务，一般为DDD中的Domain，可选
  │
  └─⫸ Commit Type: build|ci|doc|feat|fix|perf|refactor|test|...
```
其中 `type` 包含但不仅限于下：

- **build**: Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
- **ci**: Changes to our CI configuration files and scripts (example scopes: Circle, BrowserStack, SauceLabs)
- **doc**: Documentation only changes
- **feat**: A new feature
- **fix**: A bug fix
- **perf**: A code change that improves performance
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **test**: Adding missing tests or correcting existing tests


## 日志规范
日志打印的基本内容应包括：时间、env、traceId、线程、日志级别、logger、业务标识（用户id等）。

对于请求日志，需要额外打印如下信息：
- caller: 主调方
- callee: 被调方
- path: 访问路径
- response: 原始返回体
- cost: 请求耗时
- status: 返回的业务状态码
- isTimeout: 是否超时

需要注意如下的关键点：

- 打日志前思考下这行日志是不是真的对排查问题会有帮助，是否完全包含了足够的信息，不要打无意义的日志
- 描述性内容全小写，除了用于精准表达代码、类、方法名等情况
- 对于自己业务代码，按需打印方法入参、方法返回值、中间debug信息
- 对于外部服务调用，一定在调用前后分别打印请求参数、原始返回体、请求耗时
- 对于异常，一定要打印完整堆栈信息，如果需要包装异常，一定要打印log原始的错误
- 配置加载前后一定要打印日志
- 定时任务执行前后一定要打印日志
- 积累一套自己熟悉的关键词： `call` / `done call` / `request xxx, req: {}` / `done request xxx, rsp: {}, cost: 123ms` / `error when xxx` / `using` / `metrics` / `audit` / `access` / `response` 
- access日志维持特定格式，方便后期解析分析与统计。例如：统计近七天访问下游某服务的平均耗时与超时比例
- 预留 API 动态调整日志级别
- 对于Java框架中的日志，记得补充线程池调用缺少的MDC信息，例如：
```java
final Map<String, String> mainMdcContext = MDC.getCopyOfContextMap();	// org.slf4j.MDC
try {
    Future<Object> t1 = networkIoExecutor.submit(() -> {
        MDC.setContextMap(mainMdcContext);
        // biz logic here ...
        return null;
    });
    Future<Object> t2 = networkIoExecutor.submit(() -> {
        MDC.setContextMap(mainMdcContext);
        // Biz logic here ...
        return null;
    });

    t1.get(5, TimeUnit.SECONDS);
    t2.get(5, TimeUnit.SECONDS);
} catch (Exception e){
    LOGGER.error("error when exec completable futures.", e);
    throw new ApiException(CommonException.INVOKE_FAILED);
}
```

日志模板 sample log4j
```xml
<Property name="pattern.cls">[%d{yyyy-MM-dd HH:mm:ss.SSS}][%X{env}][%X{service}][%X{traceId}][%-5level][%logger{3}][%X{channel-id}][%X{wecar-id}]-%notEmpty{[optionalField=%X{optional}]}%replace{%msg %rEx{36}}{[\r\n]+}{ }%ex{0}%n</Property>
```

// todo - 可以实现一个工具去动态的添加日志的各个字段（必填、选填、默认）并生成解析用的正则表达式、MDC工具类

Ref: [最佳日志实践（v2.0）](https://zhuanlan.zhihu.com/p/27363484)

## 监控规范
- 健康检查
- CPU: 70%
- 内存: 80%
- 服务重启（Pod销毁重建）: 每次
- 业务的四个黄金指标
    - 延迟
    - 吞吐量
    - 错误
    - 饱和度（CPU / MEM）


# 使用四层（五层）模型或六边形模型而非三层模型

- 四层模型：
   - Controller：参数提取、校验、返回结构组装
   - Service：本服务（核心业务逻辑）与外部服务（通知、订单等）与服务间的协调调用
      - **Biz：核心业务逻辑，仅由Service调用**
   - **[ Repo ]：可选，组装不同DAO读取的数据实体**
   - Dao：原始的数据输入组件
- 六边形架构：领域对象位于中心，周围是 `Input / Output Adapter`，所有除了领域核心逻辑的部分都是在「适配」外部输入、输出，如下图：
  
  ![image.png](https://cdn.nlark.com/yuque/0/2020/png/657413/1603939654655-7352f2a6-2274-4a16-ad01-638c60986564.png#align=left&display=inline&height=752&margin=%5Bobject%20Object%5D&name=image.png&originHeight=752&originWidth=1100&size=947781&status=done&style=none&width=1100)

其核心要义在于，Biz 或者领域对象，一定可以独立于框架（Spring等）存在，并且可以独立的进行测试。四层模型也是六边形架构的一种特例，可以看做只包含一个输出适配器（转换成controller可用的VO）和一个输入适配器（转换成dao可用的PO）的六边形架构。

不同与上述思想，使用函数式编程：通过数据管道操作同一个对象，此时需要保证每一个管道组成部分都可以独立运作并测试。


# 多源对象需要使用仓库（Repository）封装为领域对象
在从多个数据源读取对象的数据时，需要使用repository来封装这些与数据源交互的细节，我们常用的访问db的dao应该是repository的一个组件，与dao相似的是，redis读取的逻辑也应该封装到repository中。

例如，实现一个拉取文章列表 `listArticle` 的逻辑，每个文章除了自身信息存储在 db 中外，还有点赞等信息存储在 redis 中，此时我们将 `articleDao.listById` 和 `articleRedisUtil.loadLikeNum` 读取点赞数的逻辑封装在  `articleRepo` 的子方法中，一是方便在通过不同方式（例如 `loadById` 等等）读取文章时能复用读取点赞的逻辑，二是屏蔽了对象底层的存储细节，避免  `infrastructure` 的概念泄漏到 `service` 层中。

同理，如果领域模型需要发布时间，发布的逻辑同样应当包含在 `articleRepo` 中，模型做的事情就是单纯的把 `event` 存储到自身的 `events` 字段中。

需要强调的是，repo 不应该按照 DDD 的理想情况去极端地仅实现 `load()` 和 `save()` 两个方法。由于 save 是以聚合根为单位的，这样实现会产生严重的性能问题。实践中可以根据不同的情况实现不同的 `load()`，并增加一个 `update(obj, diff)` 用于增量更新领域对象某些字段。

Repository 正如其名，其本质是领域对象的仓库，是领域对象和基础设置之间的桥梁。


# Repo 自洽
理想情况下，一个项目的 Git Repo 应该是自洽的，即：Repo 本身包含原始产品需求文档、项目自身源码、开发文档、构建脚本等该工程涉及到的所有内容。典型的错误：Git Repo 中只存源码，文档在第三方 Wiki，构建工具在第三方 CI 平台。

工作和生活的文件夹也应该如此，一个 Project 文件夹应该是该 Project 涉及到的所有内容，包括个人备忘、业务资料、代码等。


# 常用的几组概念
这些概念并不是完全对立，他们都有其各自的优缺点，要看情况决定怎么使用；有时候也需要同时使用多种方法。


- 同步、异步
- 阻塞、非阻塞
- 推、拉
- 全量、增量
- client-side / server-side
- 命令式、声明式
- 读、写：读多写少、重写轻读
- 动态数据、静态数据
- 冷数据、热数据



# 技术的目的

- 高性能：副本（replication）、分片（sharding）、一致性协议、IO 复用、零拷贝、MQ
- 容错：副本（replication）、一致性协议
- 解耦：MQ、设计模式
- 管理复杂度、封装：面向对象编程、设计模式（编程、微服务、云原生）、容器化、容器编排系统、包管理
- 提升可观测性，快速定位异常：日志系统（Logging）、链路追踪系统（Tracing）、监控系统（Monitoring）
- 快速迭代：测试、CI / CD



# 开发的几个阶段
每个阶段都可以准备相应的模板，大大提高效率

- 明确需求
- 系统设计：方法论
- 编码开发：项目代码模板
- 功能测试：
   - 预留内部接口实现特定功能
- 上线：
   - 预留曳光弹接口，检测下游组件稳定性
   - 维护 CI / CD 模板
- 运维与监控：维护日志模板、预留性能指标接口
- 性能优化：方法论

这其中需要准备如下这些资源，均需至少两套，对应正式环境和测试环境：

- 部署环境：K8s Workloads、DB、MQ
- 流水线：持续集成流水线
- 服务发现注册：如果非服务自注册，则需要配置服务发现
- 日志上报：正式与测试环境共用一套即可，方便快速定位是否为访问环境错误的问题
- 监控告警


# 解耦的核心是隔离可变性，让代码 Easy to Change
在任何可能发生变化的位置进行「封装」，具体方法就是使用适配器或分层等，将「变化」限制在封装的范围内。


# 开发前要想好如何测试代码
不一定严格追求TDD，TDD的最大优势也不是「测试」本身， 而是可以让开发人员在开发的过程中思考如何测试代码，进一步促成保护性变成和解耦。不易测试的代码一定不是松耦合的代码。


# 为什么要写好代码和测试

1. 个人追求，除了此项，后面的都很功利
1. 支持需求快速开发和变更，好的代码总是 easy to change 的；好的测试让代码变更没有心理压力
1. 支持快速定位问题，快速拿出日志证据，避免扯皮浪费时间。不逃避责任，但拒绝飞来横锅
1. 接口错误异常简洁易懂，避免联调人员反复人工确认，耗散精力
1. 文档完善严谨，理由同上



# 曳光弹模式
对于复杂的业务逻辑实现，例如访问 A 服务后、再访问 B 服务、再访问DB，先实现一系列小代码保证 A、B、DB 都是可以成功访问的，再实现具体的整合逻辑。


# 不可靠时钟
时钟有两种：时钟和单调钟。前者适用于获取当前绝对时间，后者适用于衡量时间流逝。


通常我们使用的 `System.currentTimeMillis()` 是本机的系统时钟，有可能因为 `NTP(The Network Time Protocol)` 发生时间回退的问题，导致不可靠的分布式系统异常，例如「最后写入为准」中的时间戳实际上并非分布式系统中全局一致的时间戳。与之相对的， `System.nanoTime` 是「单调钟」，他的绝对值没有意义，但其保证了自增的性质，使用与时间长度的计算，例如请求耗时、计算耗时。


# 领域对象需实现 Java Bean 规范的 getter、setter、无参构造器

虽然完全暴露 getter 和 setter 对于领域对象而言是「错误」的，但是这 getter / setter 是整个 Java 生态都遵循的的 Java Bean 规范，为了保证各种组件（JSON、ORM等）都能正常完整的访问到对象，还是要注意完全实现所有的 getter / setter。


举个例子：未实现完整 getter / setter 时，通过 Jackson 序列化后缓存到 redis 中，可能导致序列化字段不完整，进一步导致返回的反序列化数据不完整。
