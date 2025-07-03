# 20250701 题 1----------------------------------------------------------------------------------------------------------
### Java & AI 开发高级面试题（含答案）

---rb

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
8
---

**20. Java 8 中的 Lambda 表达式和 Stream 有哪些应用？**

**答案：**

* Lambda：简化匿名类，常用于函数式接口（Runnable、Comparator）
* Stream：链式操作集合，支持 map、filter、reduce、collect 等函数式操作

---

如需继续扩展并发、JVM、设计模式、集合源码分析等方向，可继续追加。

### Java 基础面试题（共 40 道，含答案）

---

#### ✅ 第一部分（1-20 题，略）

👉 请见上文，已整理

---

#### ✅ 第二部分（21-40 题）

**21. Java 中常见的集合类有哪些？它们的特点？**

**答案：**

* List：有序可重复（ArrayList、LinkedList）
* Set：无序不重复（HashSet、TreeSet、LinkedHashSet）
* Map：键值对结构（HashMap、TreeMap、ConcurrentHashMap）

---

**22. HashMap 的底层实现原理是什么？**

**答案：**
JDK 1.8 采用数组 + 链表 + 红黑树结构：

* 哈希值确定数组索引（桶）
* 桶中有多个键值对，冲突时用链表存储
* 链表超过一定长度（8）后转为红黑树，提升查找效率

---

**23. Java 中的自动装箱与拆箱是什么？可能带来哪些问题？**

**答案：**

* 自动装箱：基本类型 → 包装类
* 自动拆箱：包装类 → 基本类型

⚠️ 问题：容易导致 NPE，如 Integer i = null; int j = i;

---

**24. String 常量池是如何工作的？**

**答案：**
JVM 在方法区维护一个字符串常量池，对相同内容的字符串只保存一份（提高性能，节约内存）。`String.intern()` 可将字符串放入常量池。

---

**25. 为什么 String 是不可变的？**

**答案：**

* 安全性（如在网络类中）
* 缓存性（可放入常量池复用）
* 线程安全（不变更）

---

**26. 什么是类加载过程？**

**答案：**
包括 5 个阶段：

* 加载（Load）
* 验证（Verify）
* 准备（Prepare）
* 解析（Resolve）
* 初始化（Initialize）

---

**27. 双亲委派模型的作用是什么？**

**答案：**
防止类的重复加载，提升安全性。加载顺序：Bootstrap → Extension → AppClassLoader → 自定义 ClassLoader。

---

**28. equals 和 == 在 Integer 比较时有什么坑？**

**答案：**
Integer 在 -128 到 127 之间会缓存对象，所以用 `==` 可能返回 true，超出这个范围会 new 对象，`==` 失败，但 `equals()` 仍然为 true。

---

**29. Java 中有哪些引用类型？用途？**

**答案：**

* 强引用（默认）
* 软引用（SoftReference）：内存不足时回收，缓存常用
* 弱引用（WeakReference）：GC 会立即回收
* 虚引用（PhantomReference）：用于监控对象被 GC 回收

---

**30. 为什么 Java 不支持多继承？接口是如何解决的？**

**答案：**
为了避免菱形继承、方法冲突等问题。
接口支持多实现（从 Java 8 开始支持默认方法），通过显式 override 解决冲突。

---

**31. transient 和 volatile 有什么区别？**

**答案：**

* transient：关键字用于序列化，标记字段不参与序列化
* volatile：保证可见性，不保证原子性，适合状态标识位等场景

---

**32. instanceof 和 getClass() 区别？**

**答案：**

* instanceof 可判断子类、父类关系，支持多态
* getClass() 判断对象实际类型，严格等于当前类

---

**33. 重载和重写的区别？**

**答案：**

* 重载（Overload）：同类中方法名相同，参数不同
* 重写（Override）：子类重写父类方法，方法签名相同

---

**34. final、finally 和 finalize 的区别？**

**答案：**

* final：关键字，表示不可变
* finally：异常处理中的代码块，始终执行
* finalize：Object 的方法，GC 前调用，不建议使用

---

**35. switch 支持哪些类型？**

**答案：**

* JDK 7+：支持 String、int、char、enum、byte、short
* JDK 14+：支持 switch 表达式（返回值）

---

**36. 构造代码块、静态代码块、构造方法执行顺序？**

**答案：**
静态代码块（类加载）→ 构造代码块（每次实例化）→ 构造方法

---

**37. Java 中如何创建不可变类？**

**答案：**

1. 类用 final 修饰
2. 所有字段 private final
3. 不提供 setter
4. 提供深拷贝方法

---

**38. ArrayList 是线程安全的吗？如何改造？**

**答案：**
不是线程安全的。
改造方式：

* 使用 `Collections.synchronizedList()` 包装
* 使用 `CopyOnWriteArrayList`

---

**39. Java 中的内存泄漏有哪些常见原因？**

**答案：**

* 静态集合引用未释放
* 线程未终止
* 注册监听器未移除
* 数据库连接/流未关闭

---

**40. 你了解哪些常见的 Java 编码规范？**

**答案：**

* 类名用大驼峰，变量用小驼峰
* 常量用全大写下划线分隔
* 一个类一个文件
* 方法尽量单一职责
* 使用 javadoc 注释 API 接口

---

如需继续：可按模块继续追加“集合源码分析”、“线程并发”、“JVM机制”、“设计模式”或“手写代码题”。


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


### Java 基础面试题（共 100 道，含答案）

---


---

#### ✅ 第五部分（101-130 题）【高可用分布式架构设计】

**101. 什么是 CAP 原理？如何在实际系统中权衡？**

**答案：**
CAP 指的是一致性（Consistency）、可用性（Availability）和分区容忍性（Partition tolerance）。分布式系统无法同时满足三者，只能满足其二。常见的权衡：

* CP（如 Zookeeper）：重一致性，牺牲部分可用性
* AP（如 Redis Cluster）：高可用，但不保证强一致性

---

**102. 什么是雪崩、穿透、击穿？如何防护？**

**答案：**

* 雪崩：缓存大批失效 → 后端被打爆
* 穿透：恶意请求不存在 key，直达 DB
* 击穿：热点 key 失效瞬间被高并发请求

防护方式：设置合理过期策略、布隆过滤器、互斥锁、预加载、限流降级等

---

**103. 你如何设计一个高可用的服务注册中心？**

**答案：**

* 主流如 Nacos、Eureka、Consul 都支持集群部署
* 通过心跳保活机制剔除不可用实例
* 可用性保障：自注册 + 客户端缓存 + 服务续约 + 自动剔除

---

**104. 分布式系统如何做全链路追踪？**

**答案：**
通过唯一 TraceId 贯穿请求，结合埋点（如 Sleuth + Zipkin/Jaeger），采集链路日志、耗时、异常信息。

---

**105. 如何设计一个异地多活系统？核心难点？**

**答案：**
核心目标是实现跨地访问最优、副本独立承载业务。
难点包括：数据同步延迟、冲突解决、用户定位就近访问、状态协同。
常见技术：多活架构 + 灾备切换 + 双写一致性策略 + 跨域路由。

---

**106. 如何设计一个支持动态扩缩容的微服务系统？**

**答案：**

* 容器化部署（K8s）
* 服务无状态化
* 配合 HPA/VPA 自动扩缩容
* 使用配置中心调整线程/连接池等参数

---

**107. 分布式限流方案有哪些？**

**答案：**

* 本地限流（Guava、令牌桶）
* 分布式限流（Redis + Lua、Sentinel、Nginx）
* API 网关限流
* 接入层或边缘节点限流

---

**108. Redis 如何做高可用？**

**答案：**

* 哨兵模式 Sentinel（主从 + 自动故障转移）
* Redis Cluster（分片 + 高可用）
* 外部代理（如 Codis、Twemproxy）
* 持久化保障：RDB + AOF

---

**109. 消息队列如何保障消息不丢失？**

**答案：**

* 生产者端确认机制（事务消息）
* Broker 端持久化、主从复制
* 消费者端幂等消费 + ACK + 重试机制

---

**110. 微服务之间如何保持数据一致性？**

**答案：**

* 本地事务 + 补偿机制（Try-Confirm-Cancel）
* 可靠消息最终一致性
* 分布式事务协议：2PC、3PC、SAGA

---

**111. 怎么设计一个支持幂等性的接口？**

**答案：**

* 请求唯一标识（如全局 ID）
* 服务端记录处理结果或状态
* 使用 Redis + SETNX 保证只处理一次

---

**112. 高并发下如何避免数据库写入压力？**

**答案：**

* 缓存预写入
* 异步落库（消息队列 + 批量入库）
* 分库分表，水平扩展
* 业务去重逻辑防止重复写入

---

**113. 微服务中如何实现灰度发布？**

**答案：**

* 按用户、IP、版本标签做流量分流
* 配合服务网关（Nginx、Istio）路由策略
* 支持快速回滚、熔断

---

**114. 怎么设计一个支持多租户的 SaaS 系统？**

**答案：**

* 数据隔离（数据库、表、字段隔离）
* 权限隔离（用户、租户、角色）
* 配置隔离（租户特有参数）
* 可扩展租户定制插件

---

**115. 怎么实现服务调用链的监控和告警？**

**答案：**

* 使用 APM 工具（Skywalking、Pinpoint、Prometheus）
* 采集调用时间、失败率、慢调用等指标
* 结合 Prometheus + Alertmanager 触发告警

---

**116. 什么是服务雪崩？如何预防？**

**答案：**
雪崩：下游服务不可用，引发连锁故障。
防护：熔断（Hystrix、Sentinel）、隔离（线程池/信号量）、降级、限流。

---

**117. 如何实现配置的动态刷新？**

**答案：**

* 使用配置中心（Nacos、Apollo）
* Spring Cloud Config + Bus 机制推送
* 本地监听配置文件变化触发 reload

---

**118. Kafka 如何保证数据不重复消费？**

**答案：**

* 开启幂等生产
* 设置合适的 offset 提交策略
* 消费端记录消费位点，幂等消费

---

**119. 微服务如何实现统一认证授权？**

**答案：**

* 中央认证服务（如 OAuth2 Server）
* JWT Token 鉴权 + 网关统一拦截
* 每个服务通过 Token 验证用户权限

---

**120. 如何设计一个异步任务调度平台？**

**答案：**

* 支持任务定义 + 时间规则（Quartz/xxl-job）
* 多节点分布式调度（ZK、DB、Redis 锁）
* 支持状态跟踪、失败重试、告警、可视化

---

**121. 分布式环境中如何保证定时任务不重复执行？**

**答案：**

* 使用数据库、Redis、Zookeeper 做分布式锁
* 选主节点执行，其余节点监听
* 使用调度平台自带的集群调度能力

---

**122. 什么是服务治理？包含哪些核心能力？**

**答案：**
包括注册发现、负载均衡、熔断降级、配置管理、链路追踪、权限控制、限流治理、路由控制等。

---

**123. 如何实现接口级别的流量控制？**

**答案：**

* Sentinel 热点参数限流
* API 网关限流配置
* 本地缓存计数器
* 自定义注解 + AOP 控制频率

---

**124. 分布式事务失败如何补偿？**

**答案：**

* 使用 TCC 模型中的 Cancel
* 补偿型消息机制：业务失败时记录日志、重放消息
* 人工审核 + 运维工具补偿失败数据

---

**125. 微服务间调用失败如何自动重试？**

**答案：**

* Feign 的 Retryer 功能（默认关闭）
* 业务幂等保证前提下配置重试次数和间隔
* 使用异步补偿机制处理最终失败情况

---

**126. 如何设计一个全局唯一 ID 生成系统？**

**答案：**

* 雪花算法（时间戳 + 机器 ID + 序列）
* 数据库自增（有单点）
* Redis INCR
* Leaf、UidGenerator 等组件

---

**127. 在分布式系统中如何防止重复提交？**

**答案：**

* Token 机制（接口防重令牌）
* Redis 实现幂等判断 + 过期策略
* 数据库唯一约束

---

**128. 如何实现服务的弹性伸缩？**

**答案：**

* 基于监控指标触发（CPU、QPS）
* 使用 K8s HPA、阿里 EDAS 等平台
* 支持水平伸缩（副本数）、垂直伸缩（资源规格）

---

**129. 如何防止服务注册异常导致全局不可用？**

**答案：**

* 服务端缓存注册数据，非强依赖实时中心
* 客户端缓存（如 Ribbon、Spring Cloud）
* 引入注册中心多机房部署 + 热备份

---

**130. 如何实现异地容灾和快速切换？**

**答案：**

* 主备数据同步（异步复制 / Canal）
* 业务层隔离逻辑（配置文件切换、路由层）
* 域名 + DNS 轮询、流量调度策略实现快速切流

---

👉 如需继续，下一模块可以扩展为：DevOps、观测平台、LangChain/RAG 实战、Agent 架构、AIGC 工程化等。

---

#### ✅ 第六部分（131-135 题）【10 亿级用户系统设计面试题】

**131. 如何设计一个支持 10 亿用户的登录系统？**

**参考答案：**

* **服务拆分**：将登录鉴权逻辑与用户资料、权限解耦
* **数据库策略**：分库分表 + 用户 ID hash 分片；热数据放 Redis
* **Token 管理**：JWT 或 Session + Redis 存储
* **中间件支持**：限流（Guava/Sentinel），黑名单拦截
* **安全策略**：验证码、短信验证、多因子认证、防撞库

---

**132. 如何设计一个支持 10 亿用户访问的评论与点赞系统？数据库该怎么建模？**

**参考答案：**

* **评论表建模**：comment\_id，post\_id，user\_id，时间戳，内容等
* **点赞存储方式**：Redis Set 或 Bitmap，提高空间利用率
* **分库分表策略**：按 post\_id 或 user\_id 分表，避免单表数据过大
* **缓存优化**：热门帖子评论/点赞计数缓存 + 异步刷新 DB
* **数据一致性处理**：异步任务 + 日志补偿

---

**133. 如何设计一个全球用户低延迟访问的系统？**

**参考答案：**

* **CDN 加速**：静态资源分发（图片、视频）
* **就近接入**：通过 GeoDNS、Anycast 路由至最近区域服务节点
* **异地多活部署**：多机房部署，用户请求本地实例
* **数据同步机制**：MySQL 双写/异步 MQ 同步、Canal、Debezium
* **多活一致性设计**：双写校验、冲突检测与补偿策略

---

**134. 如何在 10 亿用户下做限流与降级设计？**

**参考答案：**

* **限流手段**：接口限流（令牌桶、滑动窗口）、网关限流、Sentinel 控制 QPS
* **降级策略**：缓存兜底、熔断逻辑、非核心服务临时关闭
* **架构层限流位置**：前端 → 网关 → 应用 → 服务层 → DB 层，逐层限流
* **用户分级保障**：VIP 优先保障，普通用户排队等待

---

**135. 如何设计支持 10 亿级用户的数据库架构？**

**参考答案：**

* **数据库高可用**：主从复制、MGR、分布式数据库（TiDB）
* **水平扩展策略**：逻辑分库 + 分表 + 路由层（ShardingSphere）
* **缓存前置**：Redis/Memcached 缓解读压力，热点数据预热
* **索引优化**：复合索引 + 分区索引，避免全表扫描
* **冷热数据分层**：冷数据归档、日志型表做分区
* **数据迁移**：使用 Canal + 增量数据同步平台实现不停机扩容

---

👉 下一模块建议：LangChain 实战 + RAG 架构 / 项目架构图手绘题 / AIGC 工程化问题。如需导出或生成 Word/PDF，请告诉我。



