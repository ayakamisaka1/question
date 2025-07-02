# 20250701 题 1----------------------------------------------------------------------------------------------------------
### Java & AI 开发高级面试题（含答案）

---

#### 一、Java基础 & JVM 优化

**1. Java 中的 volatile 关键字的作用是什么？能替代锁吗？**

**答案：**
volatile 可保证变量的可见性（线程修改的值立即对其他线程可见），但不能保证原子性，因此不能完全替代锁。在并发编程中常与 CAS 搭配使用实现轻量级线程安全。

**2. JVM 中有哪些常见的垃圾回收器？各自适用于哪些场景？**

**答案：**

* Serial：适用于小内存单核场景
* Parallel（吞吐量优先）：适合 CPU 资源紧张、对响应不敏感的批处理任务
* CMS（低延迟优先）：适合用户交互场景，低停顿
* G1：适合大内存、多核、对停顿有要求的服务
* ZGC/ Shenandoah：低延迟高可扩展，适用于超大堆

**3. JVM 调优一般怎么做？**

**答案：**

* 设置合适的堆大小、年轻代、老年代比例
* 使用 GC 日志分析工具（如 GCViewer）查看停顿
* 尽量减少 Full GC 次数
* 选择合适 GC 策略（如 G1）
* 使用逃逸分析，开启 `-XX:+UseStringDeduplication` 等

**4. 场景题：某接口响应时间突然变慢，你如何快速排查是否是 JVM 问题？**

**答案：**

1. 检查 GC 日志是否频繁发生 Full GC。
2. 使用 `jstat` 或 Arthas 查看堆使用情况。
3. 导出 heap dump 结合 MAT 分析内存泄漏。
4. 查看线程堆栈判断是否有死锁或等待 GC。

---

#### 二、Spring 全家桶 & Spring AI

**5. Spring Bean 的生命周期有哪些阶段？**

**答案：**
创建（Constructor）→ 属性填充 → 初始化（InitializingBean, @PostConstruct）→ 使用 → 销毁（DisposableBean, @PreDestroy）

**6. 如何在 Spring 中实现自定义 Starter？**

**答案：**

* 定义自动配置类并加 `@Configuration` 和 `@ConditionalOnXXX`
* 编写配置属性类 `@ConfigurationProperties`
* 在 `META-INF/spring.factories` 中注册自动配置类

**7. Spring AI 的核心能力有哪些？**

**答案：**

* Prompt Template 支持
* RAG (检索增强生成) 能力
* Function Calling 与工具集成
* LLM 接口统一封装，支持 OpenAI、Azure、Ollama、Cohere 等

**8. 场景题：如何将 LangChain + Spring AI 融入已有系统？**

**答案：**

1. 使用 Spring AI 提供的 `PromptTemplate` 和 `ChatClient` 接入 LLM。
2. 利用 LangChain 的 RetrievalChain 做文档问答。
3. 将向量检索服务封装为独立微服务。
4. 设置统一的提示词模板和缓存控制策略。

---

#### 三、数据库：MySQL / Oracle / PostgreSQL

**9. 如何优化一个慢查询？**

**答案：**

* EXPLAIN 分析执行计划
* 添加合适索引，避免全表扫描
* 拆分大 SQL 为多个小 SQL
* 避免 SELECT \*，使用覆盖索引
* MySQL：避免在 WHERE 中对字段进行函数操作

**10. Oracle 中常用的性能优化工具有哪些？**

**答案：**

* AWR、ASH 报告分析
* SQL Tuning Advisor
* SQL Profile & Plan Baseline

**11. PostgreSQL 中 JSON 类型和全文检索如何使用？**

**答案：**

* JSON: 使用 `->`, `->>`, `jsonb_path_query` 等函数
* 全文检索: 使用 `tsvector`, `tsquery`, `to_tsvector`, `@@`

---

#### 四、AI / LangChain / RAG / Prompt Engineering

**12. LangChain 的核心组件有哪些？**

**答案：**

* Chains（链式调用）
* Agents（自主调用工具）
* Tools（调用插件/函数）
* Memory（记忆存储）
* VectorStore（向量库存储）

**13. 什么是 RAG？它的典型流程是什么？**

**答案：**
RAG（Retrieval-Augmented Generation）是结合外部知识库进行检索后再生成的 AI 方法。
流程：输入 -> Embedding -> 检索 (向量数据库) -> 拼接上下文 -> LLM 响应

**14. 提示词优化有哪些技巧？**

**答案：**

* 明确角色（You are a senior Java engineer）
* 结构化输出（JSON, Markdown）
* 添加限制（限制字数、输出格式）
* Chain-of-thought 逐步引导
* 使用系统提示词提升控制能力

---

#### 五、消息中间件

**15. Kafka 和 RabbitMQ 的区别？**

**答案：**

* Kafka 是分布式日志，偏向高吞吐、高可扩展，适用于大数据、日志
* RabbitMQ 是 AMQP 实现，可靠性高、支持丰富消息模型，适用于业务解耦

**16. 如何实现消息幂等性？**

**答案：**

* 使用唯一消息 ID + Redis 去重
* 数据库唯一约束插入
* 利用消费端记录消费日志

**17. RocketMQ 中的事务消息是怎么实现的？**

**答案：**
发送方发送 prepare 消息 -> 执行本地事务 -> 根据事务执行结果发送 commit/rollback 消息 -> Broker 最终确认消费

**18. 场景题：Kafka 消费延迟越来越大如何排查？**

**答案：**

1. 查看 Lag（滞后量）判断是消费速度慢还是生产速度快。
2. 检查消费逻辑是否慢（阻塞/IO 等）。
3. 增加分区或并行消费线程。
4. 分析 GC 或 CPU 是否瓶颈。

---

#### 六、搜索引擎与缓存

**19. Elasticsearch 中的倒排索引如何工作的？**

**答案：**
倒排索引建立词项到文档的映射，能快速定位包含某个词的所有文档，是全文搜索引擎的核心结构。

**20. Redis 如何实现分布式锁？**

**答案：**

* 使用 SET key value NX PX 实现
* 要加锁成功必须确保唯一性
* 使用 Redisson 框架或自己实现带过期时间+续期机制

**21. ClickHouse 的数据写入机制是怎样的？适合哪些场景？**

**答案：**
ClickHouse 是列式存储，适合 OLAP 查询场景。写入数据是批量插入，并在后台合并，避免频繁 I/O。

---

#### 七、设计模式 & 流程引擎 & 编排

**22. 工厂模式与策略模式区别？**

**答案：**
工厂模式负责创建对象，策略模式封装行为算法并可以替换。
工厂是创建型，策略是行为型。

**23. 什么是流程引擎？如何选型？**

**答案：**
流程引擎是一种业务流程驱动机制，常见如 Camunda、Flowable、Activiti。选型看：

* BPMN 支持度
* 易用性（Web建模）
* 扩展性与 Java 集成能力

**24. 编排和编排引擎的区别？**

**答案：**
编排是对多个服务、工具或 Agent 的逻辑流程设计，编排引擎提供任务调度与状态追踪能力。常见工具如 Airflow、Temporal、Langflow 等。

**25. 场景题：你如何设计一个用于多 Agent 协作的编排平台？**

**答案：**

* 使用 Webhook 与 Agent 通信
* 定义任务状态机 + 状态追踪表
* 结合 RAG+LangChain 作为智能节点
* 使用 Camunda/Temporal 控制流程

---

#### 八、AI 微调 & MCP & Agent

**26. 什么是 LoRA 微调？它的优点是什么？**

**答案：**
LoRA（Low-Rank Adaptation）通过低秩矩阵调整模型部分权重，参数量少、训练快、可迁移性强。

**27. 什么是 MCP 协议？它解决了什么问题？**

**答案：**
MCP（Multi-component Protocol）是 AI 多组件交互协议，解决了 LLM + 工具链之间结构化、顺序性通信问题。

**28. Agent 中如何使用工具调用？**

**答案：**
通过 Tool（工具描述）+ Function Calling 机制，LLM 可以自动判断并调用注册工具，LangChain/Autogen 都提供了实现框架。

---

#### 九、补充问题（跨语言 & Python）

**29. Python 中 GIL 的作用与缺点是什么？**

**答案：**
GIL（全局解释器锁）保证线程安全，但限制了多线程并行执行计算任务，适合 I/O 密集型任务，多进程可解决此问题。

**30. Python 与 Java 在 AI 应用开发上如何互补？**

**答案：**
Python 适合模型调用、快速原型，Java 适合工程化与生产部署，可通过 HTTP/Socket/RPC/ONNX 等方式互通。

---

> 本题库内容可根据候选人实际背景动态调整，适合技术总监 / 架构师 / 高级开发面试使用。


# 20250701 题3---------------------------------------------------------------------------------------------------
### Java 基础面试题（20 道，含答案）

---

**1. Java 中的基本数据类型有哪些？包装类型是什么？**

**答案：**
基本类型：byte、short、int、long、float、double、char、boolean，共 8 个。
包装类型：Byte、Short、Integer、Long、Float、Double、Character、Boolean。

---

**2. == 和 equals() 有什么区别？**

**答案：**

* `==` 比较的是对象的地址（引用是否相同）
* `equals()` 默认实现也是比较地址（Object），但很多类（如 String、Integer）重写了 equals()，改为比较内容

---

**3. String、StringBuilder 和 StringBuffer 有什么区别？**

**答案：**

* String 是不可变类，每次修改都会生成新对象
* StringBuilder 是可变类，线程不安全，性能高
* StringBuffer 是线程安全的，适用于并发场景

---

**4. Java 中的值传递和引用传递是什么？Java 是哪种？**

**答案：**
Java 是**值传递**，传递的是“值的副本”：

* 对象传递的是引用的副本（即地址的副本），所以能修改对象内容
* 基本类型传的是值的副本，修改不影响原值

---

**5. final 和 static 的区别？**

**答案：**

* final：修饰变量（不可变）、方法（不可重写）、类（不可继承）
* static：属于类，不属于实例，可修饰变量、方法、代码块、内部类

---

**6. 抽象类和接口的区别？**

**答案：**

* 抽象类可以有构造方法、成员变量和非抽象方法
* 接口只能定义方法签名（Java 8 之后支持默认方法和静态方法）
* 一个类只能继承一个抽象类，但可实现多个接口

---

**7. Java 中异常的分类？区别？**

**答案：**

* CheckedException（编译时异常）：必须 try-catch，例如 IOException、SQLException
* UncheckedException（运行时异常）：如 NullPointerException、IndexOutOfBoundsException

---

**8. try-with-resources 是什么？有什么好处？**

**答案：**
Java 7 引入的语法，自动调用资源的 close() 方法，避免资源泄漏。
适用于实现了 AutoCloseable 接口的类。

---

**9. equals() 和 hashCode() 有什么关系？**

**答案：**

* 若两个对象 equals() 相等，hashCode() 必须相同
* 若两个对象 hashCode() 相等，不一定 equals()
* 在集合类如 HashMap 中依赖 hashCode() + equals() 判等

---

**10. 集合类 List、Set、Map 的区别？**

**答案：**

* List：有序、可重复，如 ArrayList、LinkedList
* Set：无序、不可重复，如 HashSet、TreeSet
* Map：键值对结构，key 不重复，如 HashMap、TreeMap

---

**11. ArrayList 和 LinkedList 区别？**

**答案：**

* ArrayList 底层是数组，查询快、增删慢
* LinkedList 是链表结构，查询慢、增删快

---

**12. HashMap 和 ConcurrentHashMap 的区别？**

**答案：**

* HashMap 是线程不安全的
* ConcurrentHashMap 是线程安全的，采用分段锁或 CAS 操作提高并发性能

---

**13. Java 中线程的创建方式有哪些？**

**答案：**

1. 继承 Thread 类
2. 实现 Runnable 接口
3. 使用 Callable + Future
4. 使用线程池 ExecutorService

---

**14. synchronized 的作用是什么？可以作用在哪些位置？**

**答案：**
用于同步线程访问资源，可作用在：

* 实例方法（锁对象）
* 静态方法（锁类）
* 代码块（锁任意对象）

---

**15. 什么是线程安全？如何保证？**

**答案：**
多个线程访问同一资源时，数据一致、结果正确。
保证方式：

* synchronized
* Lock 显式锁
* 原子类（AtomicInteger）
* 线程安全容器（ConcurrentHashMap）

---

**16. volatile 是什么？能替代锁吗？**

**答案：**
volatile 保证变量对所有线程的可见性，不能保证原子性。
不能完全替代锁，可配合 CAS 使用。

**扩展：**
CAS（Compare-And-Swap）是一种乐观锁机制，利用原子操作保证并发安全。

---

**17. 什么是反射？反射有哪些典型用途？**

**答案：**
反射可以在运行时动态获取类的信息、调用其方法或修改字段。
典型用途：框架底层实现（如 Spring）、动态代理、插件机制。

---

**18. 泛型中的 ?、T、E、K、V 各代表什么？**

**答案：**

* T：Type（类型）
* E：Element（元素）
* K：Key（键）
* V：Value（值）
* ?: 通配符，表示未知类型

---

**19. 什么是枚举？枚举的优势？**

**答案：**
枚举是一种特殊的类，表示一组常量。
优势：类型安全、可读性高、可定义方法和字段、可用于 switch。

---

**20. Java 8 中的 Lambda 表达式和 Stream 有哪些应用？**

**答案：**

* Lambda：简化匿名类，常用于函数式接口（Runnable、Comparator）
* Stream：链式操作集合，支持 map、filter、reduce、collect 等函数式操作

---

如需继续扩展并发、JVM、设计模式、集合源码分析等方向，可继续追加。



# 20250702 题3---------------------------------------------------------------------------------------------------
### Java 基础面试题（40 道，含答案）


#### ✅ 第四部分（61-100 题）【集合源码 / 并发编程 / JVM 优化 / Spring 源码】

**61. ArrayList 的扩容机制？**

**答案：**
初始容量为 10，扩容时变为原容量的 1.5 倍（`oldCapacity + (oldCapacity >> 1)`），复制到新数组。

---

**62. LinkedList 的底层结构？**

**答案：**
双向链表，每个节点包含前驱、后继指针。
增删操作性能好，查询慢。

---

**63. HashSet 为什么不能重复？底层实现？**

**答案：**
基于 HashMap 实现，元素作为 key，value 为常量对象。
依赖 hashCode() 和 equals() 判重。

---

**64. ConcurrentHashMap 如何实现并发安全？**

**答案：**
JDK1.8 采用数组 + 链表/红黑树 + CAS + synchronized。
桶内加锁，减少锁粒度，提升并发性能。

---

**65. CopyOnWriteArrayList 的原理？适用场景？**

**答案：**
写时复制：写操作时复制数组副本，修改后替换原数组。适用于读多写少的并发场景。

---

**66. BlockingQueue 有哪些实现类？**

**答案：**

* ArrayBlockingQueue
* LinkedBlockingQueue
* PriorityBlockingQueue
* DelayQueue
* SynchronousQueue

---

**67. 线程状态有哪些？分别表示什么？**

**答案：**

* NEW
* RUNNABLE
* BLOCKED
* WAITING
* TIMED\_WAITING
* TERMINATED

---

**68. 什么是可重入锁？如何实现？**

**答案：**
同一线程可以多次获得锁而不死锁。
如：`synchronized` 和 `ReentrantLock` 实现。

---

**69. CAS 的 ABA 问题如何解决？**

**答案：**
通过 `AtomicStampedReference` 或版本号机制来记录修改次数。

---

**70. Java 中的并发容器有哪些？**

**答案：**

* ConcurrentHashMap
* CopyOnWriteArrayList
* ConcurrentSkipListMap
* BlockingQueue 系列

---

**71. ThreadPoolExecutor 的任务执行流程？**

**答案：**
任务执行顺序：

1. corePool 线程
2. workQueue
3. maximumPoolSize
4. 拒绝策略

---

**72. 常见线程池类型？**

**答案：**

* FixedThreadPool
* CachedThreadPool
* ScheduledThreadPool
* SingleThreadExecutor
* 自定义 ThreadPoolExecutor

---

**73. 什么是内存泄漏？JVM 中的排查方式？**

**答案：**
内存泄漏指对象不再使用但仍被引用。
排查工具：MAT、JProfiler、VisualVM、Heap Dump 分析。

---

**74. JVM 如何进行垃圾回收？**

**答案：**
使用可达性分析（GC Root）判断对象是否可回收。
垃圾回收器：G1、CMS（已废弃）、Serial、ZGC、Shenandoah。

---

**75. 如何调优 GC？**

**答案：**
通过设置 JVM 启动参数，如：

* `-Xms/-Xmx` 设置堆大小
* `-XX:+UseG1GC`
* `-XX:+PrintGCDetails`
* 分析 GC 日志判断 FullGC 频率、停顿时间

---

**76. 什么是内存逃逸？如何避免？**

**答案：**
对象在方法外部被引用，导致无法栈上分配。
避免方法：对象只在方法内部使用。

---

**77. 方法区和堆的区别？**

**答案：**
方法区（元空间）用于存储类元信息，堆用于存储实例对象。

---

**78. Spring 中的 Bean 生命周期？**

**答案：**

1. 实例化
2. 属性赋值
3. `setBeanName`
4. `setBeanFactory`
5. `BeanPostProcessor` 前置
6. 初始化方法（init-method/@PostConstruct）
7. `BeanPostProcessor` 后置
8. 销毁方法（destroy-method/@PreDestroy）

---

**79. Spring 中的循环依赖是如何解决的？**

**答案：**
使用三级缓存（singletonObjects、earlySingletonObjects、singletonFactories）提前暴露对象引用。

---

**80. Spring AOP 是如何实现的？**

**答案：**
基于动态代理：

* JDK Proxy（接口）
* CGLIB（类）
  通过 ProxyFactory、Advisor、MethodInterceptor 实现增强。

---

**81. BeanFactory 和 ApplicationContext 区别？**

**答案：**

* BeanFactory 是基础容器，延迟加载
* ApplicationContext 是完整容器，支持国际化、事件、AOP 等

---

**82. Spring 如何实现事务管理？**

**答案：**
基于 AOP 拦截方法，利用事务管理器（如 DataSourceTransactionManager）控制连接、提交、回滚。

---

**83. @Transactional 有哪些坑？**

**答案：**

* 方法内部调用不生效（代理失效）
* 非 public 方法不生效
* 异常未抛出或被捕获不回滚

---

**84. Spring Bean 的加载顺序？**

**答案：**

* 扫描配置类 → 注册 BeanDefinition → 实例化单例 → 注入依赖 → 初始化

---

**85. Spring 的事件机制？**

**答案：**
通过 ApplicationEvent 和 ApplicationListener 发布订阅事件，支持异步和泛型事件监听。

---

**86. Spring Boot 自动配置的原理？**

**答案：**
基于 `@EnableAutoConfiguration` 和 `spring.factories` 加载配置类，通过条件注解 `@Conditional` 控制是否生效。

---

**87. Spring 中的依赖注入方式？**

**答案：**

* 构造器注入
* Setter 注入
* 字段注入（推荐避免）

---

**88. Spring 中的条件装配有哪些方式？**

**答案：**

* `@ConditionalOnClass`
* `@ConditionalOnMissingBean`
* `@Profile`
* `@Conditional` 自定义条件

---

**89. Bean 的作用域有哪些？**

**答案：**

* singleton（默认）
* prototype
* request（Web）
* session（Web）
* application
* websocket

---

**90. 如何实现 Spring 的扩展点？**

**答案：**

* BeanPostProcessor
* BeanFactoryPostProcessor
* ApplicationListener
* SmartInitializingSingleton
* ImportSelector / ImportBeanDefinitionRegistrar

---

**91. Spring Bean 初始化失败时如何处理？**

**答案：**

* 自定义 BeanPostProcessor 捕获异常
* 配置 `@Primary` 或排除冲突 Bean
* 使用 `@DependsOn` 控制加载顺序

---

**92. 如何理解 Spring 的 IoC 和 DI？**

**答案：**

* IoC：控制反转，由容器控制对象生命周期
* DI：依赖注入，对象的依赖由容器注入

---

**93. Spring 中如何实现懒加载？**

**答案：**

* `@Lazy` 注解
* 配置 BeanDefinition 的 lazy-init 属性

---

**94. Spring 中的资源加载机制？**

**答案：**
通过 ResourceLoader 加载 classpath、file、URL 等类型的资源，可用于配置文件、模板加载等场景。

---

**95. 如何自定义 Starter？**

**答案：**

1. 提供配置类
2. 使用 `@ConfigurationProperties`
3. 创建 spring.factories 或 spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

---

**96. Spring Cloud 中的服务注册与发现原理？**

**答案：**
客户端向注册中心注册服务实例，同时定时发送心跳。
客户端从注册中心获取服务列表并进行负载均衡。

---

**97. Spring Cloud 的负载均衡方式？**

**答案：**

* Ribbon（已弃用）
* Spring Cloud LoadBalancer
* 支持轮询、权重、随机等策略

---

**98. Spring Boot 如何进行健康检查？**

**答案：**
通过 Actuator 提供 `/actuator/health` 接口，集成自定义 `HealthIndicator` 实现业务级别健康检查。

---

**99. Spring 的配置文件加载顺序？**

**答案：**
优先级从高到低：

1. 命令行参数
2. application.properties/yaml
3. @PropertySource
4. 默认配置

---

**100. 如何理解 Spring 全家桶中“约定优于配置”？**

**答案：**
通过合理默认值（如端口、路径、日志等）减少配置复杂度，只在需要时显式覆盖，提升开发效率。

---

✅ 后续建议扩展方向：手写题、设计模式、微服务、消息中间件、AI 结合题、LangChain/RAG/MCP 等。







