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

---

#### 二、Spring 全家桶 & Spring AI

**4. Spring Bean 的生命周期有哪些阶段？**

**答案：**
创建（Constructor）→ 属性填充 → 初始化（InitializingBean, @PostConstruct）→ 使用 → 销毁（DisposableBean, @PreDestroy）

**5. 如何在 Spring 中实现自定义 Starter？**

**答案：**

* 定义自动配置类并加 `@Configuration` 和 `@ConditionalOnXXX`
* 编写配置属性类 `@ConfigurationProperties`
* 在 `META-INF/spring.factories` 中注册自动配置类

**6. Spring AI 的核心能力有哪些？**

**答案：**

* Prompt Template 支持
* RAG (检索增强生成) 能力
* Function Calling 与工具集成
* LLM 接口统一封装，支持 OpenAI、Azure、Ollama、Cohere 等

---

#### 三、数据库：MySQL / Oracle / PostgreSQL

**7. 如何优化一个慢查询？**

**答案：**

* EXPLAIN 分析执行计划
* 添加合适索引，避免全表扫描
* 拆分大 SQL 为多个小 SQL
* 避免 SELECT \*，使用覆盖索引
* MySQL：避免在 WHERE 中对字段进行函数操作

**8. Oracle 中常用的性能优化工具有哪些？**

**答案：**

* AWR、ASH 报告分析
* SQL Tuning Advisor
* SQL Profile & Plan Baseline

**9. PostgreSQL 中 JSON 类型和全文检索如何使用？**

**答案：**

* JSON: 使用 `->`, `->>`, `jsonb_path_query` 等函数
* 全文检索: 使用 `tsvector`, `tsquery`, `to_tsvector`, `@@`

---

#### 四、AI / LangChain / RAG / Prompt Engineering

**10. LangChain 的核心组件有哪些？**

**答案：**

* Chains（链式调用）
* Agents（自主调用工具）
* Tools（调用插件/函数）
* Memory（记忆存储）
* VectorStore（向量库存储）

**11. 什么是 RAG？它的典型流程是什么？**

**答案：**
RAG（Retrieval-Augmented Generation）是结合外部知识库进行检索后再生成的 AI 方法。
流程：输入 -> Embedding -> 检索 (向量数据库) -> 拼接上下文 -> LLM 响应

**12. 提示词优化有哪些技巧？**

**答案：**

* 明确角色（You are a senior Java engineer）
* 结构化输出（JSON, Markdown）
* 添加限制（限制字数、输出格式）
* Chain-of-thought 逐步引导
* 使用系统提示词提升控制能力

---

#### 五、消息中间件

**13. Kafka 和 RabbitMQ 的区别？**

**答案：**

* Kafka 是分布式日志，偏向高吞吐、高可扩展，适用于大数据、日志
* RabbitMQ 是 AMQP 实现，可靠性高、支持丰富消息模型，适用于业务解耦

**14. 如何实现消息幂等性？**

**答案：**

* 使用唯一消息 ID + Redis 去重
* 数据库唯一约束插入
* 利用消费端记录消费日志

**15. RocketMQ 中的事务消息是怎么实现的？**

**答案：**
发送方发送 prepare 消息 -> 执行本地事务 -> 根据事务执行结果发送 commit/rollback 消息 -> Broker 最终确认消费

---

#### 六、搜索引擎与缓存

**16. Elasticsearch 中的倒排索引如何工作的？**

**答案：**
倒排索引建立词项到文档的映射，能快速定位包含某个词的所有文档，是全文搜索引擎的核心结构。

**17. Redis 如何实现分布式锁？**

**答案：**

* 使用 SET key value NX PX 实现
* 要加锁成功必须确保唯一性
* 使用 Redisson 框架或自己实现带过期时间+续期机制

**18. ClickHouse 的数据写入机制是怎样的？适合哪些场景？**

**答案：**
ClickHouse 是列式存储，适合 OLAP 查询场景。写入数据是批量插入，并在后台合并，避免频繁 I/O。

---

#### 七、设计模式 & 流程引擎 & 编排

**19. 工厂模式与策略模式区别？**

**答案：**
工厂模式负责创建对象，策略模式封装行为算法并可以替换。
工厂是创建型，策略是行为型。

**20. 什么是流程引擎？如何选型？**

**答案：**
流程引擎是一种业务流程驱动机制，常见如 Camunda、Flowable、Activiti。选型看：

* BPMN 支持度
* 易用性（Web建模）
* 扩展性与 Java 集成能力

**21. 编排和编排引擎的区别？**

**答案：**
编排是对多个服务、工具或 Agent 的逻辑流程设计，编排引擎提供任务调度与状态追踪能力。常见工具如 Airflow、Temporal、Langflow 等。

---

#### 八、AI 微调 & MCP & Agent

**22. 什么是 LoRA 微调？它的优点是什么？**

**答案：**
LoRA（Low-Rank Adaptation）通过低秩矩阵调整模型部分权重，参数量少、训练快、可迁移性强。

**23. 什么是 MCP 协议？它解决了什么问题？**

**答案：**
MCP（Multi-component Protocol）是 AI 多组件交互协议，解决了 LLM + 工具链之间结构化、顺序性通信问题。

**24. Agent 中如何使用工具调用？**

**答案：**
通过 Tool（工具描述）+ Function Calling 机制，LLM 可以自动判断并调用注册工具，LangChain/Autogen 都提供了实现框架。

---

#### 九、补充问题（跨语言 & Python）

**25. Python 中 GIL 的作用与缺点是什么？**

**答案：**
GIL（全局解释器锁）保证线程安全，但限制了多线程并行执行计算任务，适合 I/O 密集型任务，多进程可解决此问题。

**26. Python 与 Java 在 AI 应用开发上如何互补？**

**答案：**
Python 适合模型调用、快速原型，Java 适合工程化与生产部署，可通过 HTTP/Socket/RPC/ONNX 等方式互通。

---

> 本题库内容可根据候选人实际背景动态调整，适合技术总监 / 架构师 / 高级开发面试使用。


# 20250701 题 2----------------------------------------------------------------------------------------------------------

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









