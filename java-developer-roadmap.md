# Java 开发必备知识点与技术栈大纲

---

## 一、Java 核心基础

### 1.1 Java 语言基础
- [ ] Java 发展史与版本特性（Java 8 ~ Java 21）
- [ ] JDK / JRE / JVM 区别
- [ ] 跨平台原理（编译与解释执行）
- [ ] 基本数据类型（8 种）与包装类（缓存机制）
- [ ] 自动装箱与拆箱（`valueOf` / `xxxValue`）
- [ ] 运算符、流程控制、数组

### 1.2 面向对象（OOP）
- [ ] 封装、继承、多态、抽象
- [ ] 接口 vs 抽象类（含 Java 8 default/static 方法）
- [ ] 访问修饰符（public / protected / default / private）
- [ ] `final` / `static` / `super` / `this` 关键字
- [ ] 内部类（成员/静态/局部/匿名）- 内存泄漏风险
- [ ] 方法重载（Overload）与重写（Override）
- [ ] 静态分派与动态分派（虚方法表）
- [ ] 构造器与初始化顺序
- [ ] 浅拷贝 / 深拷贝 / `Cloneable` 接口
- [ ] 组合 vs 继承设计原则

### 1.3 核心 API
- [ ] `String` / `StringBuilder` / `StringBuffer` - 不可变性、常量池
- [ ] `Object` 类 - `equals` / `hashCode` / `toString` / `clone` / `finalize`
- [ ] `BigDecimal` / `BigInteger` - 精度计算与舍入模式
- [ ] 数学类（`Math` / `Random` / `ThreadLocalRandom`）
- [ ] 日期时间（`Date` / `Calendar` / `SimpleDateFormat` 旧版 → Java 8 `LocalDate`/`LocalTime`/`LocalDateTime`/`Instant`/`Duration`/`Period`/`DateTimeFormatter`）
- [ ] `System` / `Runtime` 类

### 1.4 异常处理
- [ ] `Throwable` 体系（Error vs Exception / 受检 vs 非受检）
- [ ] `try-catch-finally` - finally 与 return 执行顺序
- [ ] `try-with-resources` - 抑制异常（Suppressed）
- [ ] 自定义异常最佳实践
- [ ] 异常链与性能开销

### 1.5 泛型（Generics）
- [ ] 泛型擦除（Type Erasure）与桥方法
- [ ] 泛型类 / 泛型接口 / 泛型方法
- [ ] 通配符：`? extends` / `? super` / 无界通配符
- [ ] PECS 原则（Producer Extends, Consumer Super）
- [ ] `TypeToken` - 运行时获取泛型类型

### 1.6 反射（Reflection）
- [ ] `Class` 对象获取（`.class` / `forName` / `getClass`）
- [ ] 反射 API：`Field` / `Method` / `Constructor` / `Annotation` / `Modifier`
- [ ] 反射性能优化（`setAccessible` / 缓存 / `MethodHandle`）
- [ ] 动态代理：JDK Proxy（`InvocationHandler`） vs CGLIB
- [ ] 反射在框架中的应用（Spring IoC / MyBatis ORM）
- [ ] 反射破坏单例

### 1.7 注解（Annotation）
- [ ] 元注解：`@Retention` / `@Target` / `@Documented` / `@Inherited` / `@Repeatable`
- [ ] `RetentionPolicy`（SOURCE / CLASS / RUNTIME）区别
- [ ] 注解处理：运行时（反射） vs 编译期（APT / `AbstractProcessor`）
- [ ] 自定义注解设计

### 1.8 IO / NIO
- [ ] BIO 体系：`InputStream` / `OutputStream` / `Reader` / `Writer`
- [ ] 装饰器模式在 IO 中的应用（`BufferedInputStream` / `DataInputStream` / `ObjectInputStream`）
- [ ] NIO 三大核心：`Buffer` / `Channel` / `Selector`
- [ ] `ByteBuffer`：直接缓冲区 vs 堆缓冲区
- [ ] `FileChannel.map` - 内存映射文件与零拷贝
- [ ] 多路复用：`Selector`（select / poll / epoll 模型）
- [ ] AIO（`AsynchronousChannel` / `CompletionHandler`）
- [ ] IO 模型：BIO / NIO / AIO / 多路复用 / 信号驱动
- [ ] 大文件处理策略
- [ ] 字符编码：ASCII / Unicode / UTF-8 / UTF-16 / GBK

### 1.9 序列化
- [ ] `Serializable` 接口与 `serialVersionUID`
- [ ] `transient` 关键字
- [ ] `Externalizable` 自定义序列化
- [ ] JSON 序列化（Jackson / Gson / Fastjson 对比）
- [ ] Protobuf / Thrift / Avro 二进制序列化方案
- [ ] JDK 序列化 vs 跨语言序列化方案选型

---

## 二、Java 集合框架（JCF）

### 2.1 Collection 体系
- [ ] 整体架构：`Collection` → `List` / `Set` / `Queue`

### 2.2 List
- [ ] `ArrayList` - 动态数组、扩容机制（1.5 倍）、随机访问
- [ ] `LinkedList` - 双向链表、插入删除
- [ ] `Vector` / `Stack` - 线程安全（已过时）
- [ ] `CopyOnWriteArrayList` - 写时复制、读多写少场景

### 2.3 Set
- [ ] `HashSet` - 基于 HashMap、去重原理
- [ ] `LinkedHashSet` - 维护插入顺序
- [ ] `TreeSet` - 基于 TreeMap、红黑树、排序
- [ ] `ConcurrentSkipListSet` - 跳表实现、有序并发

### 2.4 Queue
- [ ] `PriorityQueue` - 二叉堆、优先级队列
- [ ] `BlockingQueue` - `ArrayBlockingQueue` / `LinkedBlockingQueue` / `PriorityBlockingQueue` / `DelayQueue` / `SynchronousQueue` / `TransferQueue`
- [ ] `ConcurrentLinkedQueue` - CAS 无锁队列
- [ ] `Deque` / `ArrayDeque`

### 2.5 Map
- [ ] `HashMap` - 数组+链表+红黑树、put/get 流程、扩容（2 的幂、0.75 阈值）、线程不安全表现
- [ ] `ConcurrentHashMap` - JDK 7 Segment 分段锁 → JDK 8 CAS+synchronized 锁细化
- [ ] `LinkedHashMap` - 双向链表维护顺序、LRU 缓存实现
- [ ] `TreeMap` - 红黑树、排序
- [ ] `Hashtable` - 线程安全已过时
- [ ] `EnumMap` / `IdentityHashMap` / `WeakHashMap`

### 2.6 工具类与迭代器
- [ ] `Collections` 工具类（排序/不可变集合/同步包装）
- [ ] `Arrays` 工具类（排序/二分/转换/`asList` 坑）
- [ ] `Iterator` / `Iterable` - 快速失败（fail-fast） vs 安全失败（fail-safe）
- [ ] `Comparator` / `Comparable`

---

## 三、Java 并发编程（JUC）

### 3.1 线程基础
- [ ] 线程创建方式：`Thread` / `Runnable` / `Callable` / `FutureTask`
- [ ] 线程状态：NEW / RUNNABLE / BLOCKED / WAITING / TIMED_WAITING / TERMINATED
- [ ] 线程通信：`wait` / `notify` / `notifyAll`、`LockSupport.park` / `unpark`
- [ ] `sleep` vs `wait`、`yield` vs `join`
- [ ] 线程中断机制：`interrupt` / `isInterrupted` / `InterruptedException`
- [ ] 守护线程（Daemon）与用户线程
- [ ] 线程组（`ThreadGroup`）与应用

### 3.2 线程池
- [ ] `ThreadPoolExecutor` 七大核心参数
- [ ] 线程池工作流程：核心线程 → 队列 → 最大线程 → 拒绝
- [ ] 四种拒绝策略：`AbortPolicy` / `DiscardPolicy` / `DiscardOldestPolicy` / `CallerRunsPolicy`
- [ ] `Executors` 工厂方法缺陷（无界队列 OOM / 最大线程 Integer.MAX_VALUE）
- [ ] 线程池大小估算：CPU 密集型 / IO 密集型 / 混合型
- [ ] `ScheduledThreadPoolExecutor` - 定时/周期任务
- [ ] `ForkJoinPool` - 工作窃取、分治算法
- [ ] `CompletableFuture` - 异步编排（`thenApply` / `thenCompose` / `allOf` / `anyOf`）
- [ ] 动态线程池监控与参数调整

### 3.3 锁机制
- [ ] `synchronized` 底层实现：`monitorenter` / `monitorexit` / `MarkWord`
- [ ] 锁升级过程：无锁 → 偏向锁 → 轻量级锁（自旋锁）→ 重量级锁
- [ ] JDK 15+ 偏向锁废弃原因
- [ ] `volatile`：可见性、禁止指令重排、内存屏障（MESI 协议）
- [ ] `Lock` 接口：`ReentrantLock`（公平/非公平、可重入、可中断、超时）
- [ ] `AQS`（`AbstractQueuedSynchronizer`）：CLH 队列、`state`、模板方法
- [ ] `ReentrantReadWriteLock`：读读共享、读写互斥、写写互斥、锁降级
- [ ] `StampedLock`：乐观读、悲观读、写锁
- [ ] `Condition`：await / signal 替代 wait / notify
- [ ] 锁优化技术：锁粗化、锁消除、自适应自旋

### 3.4 同步工具类
- [ ] `CountDownLatch` - 倒计数器
- [ ] `CyclicBarrier` - 循环屏障
- [ ] `Semaphore` - 信号量限流
- [ ] `Exchanger` - 线程间数据交换

### 3.5 CAS 与原子类
- [ ] CAS 原理（`Unsafe.compareAndSwap`）
- [ ] ABA 问题与 `AtomicStampedReference` / `AtomicMarkableReference`
- [ ] 原子类体系：`AtomicInteger` / `AtomicLong` / `AtomicBoolean` / `AtomicReference`
- [ ] 高并发计数器优化：`LongAdder` / `LongAccumulator`（分段累加）
- [ ] 字段原子更新：`AtomicIntegerFieldUpdater` / `AtomicReferenceFieldUpdater`

### 3.6 ThreadLocal
- [ ] `ThreadLocal` 原理：`ThreadLocalMap`、弱引用 Key
- [ ] 内存泄漏原因与 `remove()` 必要性
- [ ] `InheritableThreadLocal` - 父子线程传值
- [ ] `TransmittableThreadLocal`（TTL）- 线程池上下文传递

### 3.7 JMM（Java 内存模型）
- [ ] 主内存 vs 工作内存
- [ ] 8 种原子操作（lock / read / load / use / assign / store / write / unlock）
- [ ] 三大特性：原子性、可见性、有序性
- [ ] `happens-before` 原则（8 条规则）
- [ ] 指令重排序（编译器 / 处理器）
- [ ] 内存屏障（LoadLoad / StoreStore / LoadStore / StoreLoad）
- [ ] 单例 DCL 与 `volatile` 必要性

### 3.8 伪共享（False Sharing）
- [ ] 缓存行（Cache Line）与 MESI 协议
- [ ] `@Contended` 注解与缓存行填充
- [ ] 避免伪共享设计

---

## 四、JVM

### 4.1 内存区域
- [ ] 堆（Heap）：新生代（Eden / S0 / S1）、老年代
- [ ] 虚拟机栈（Stack）：局部变量表、操作数栈、动态链接、返回地址
- [ ] 方法区（Method Area）：JDK 8+ 元空间（Metaspace） vs JDK 7 永久代（PermGen）
- [ ] 运行时常量池（Runtime Constant Pool）
- [ ] 字符串常量池（String Table）与 `intern()`
- [ ] 程序计数器（PC Register）
- [ ] 本地方法栈（Native Method Stack）
- [ ] 直接内存（Direct Memory）- `DirectByteBuffer` / `Unsafe.allocateMemory`
- [ ] TLAB（Thread Local Allocation Buffer）- 线程本地分配缓冲区

### 4.2 对象
- [ ] 对象创建过程：类加载检查 → 分配内存 → 初始化零值 → 设置对象头 → 执行 init
- [ ] 对象内存布局：对象头（MarkWord + Klass Pointer）→ 实例数据 → 对齐填充
- [ ] 指针压缩（`-XX:+UseCompressedOops` / `-XX:+UseCompressedClassPointers`）
- [ ] 对象访问定位：直接指针（HotSpot） vs 句柄池

### 4.3 类加载机制
- [ ] 类加载过程：加载 → 验证 → 准备 → 解析 → 初始化
- [ ] 类加载时机：主动引用（6 种） vs 被动引用
- [ ] 双亲委派模型：Bootstrap ClassLoader → Extension ClassLoader → Application ClassLoader
- [ ] 双亲委派优势：安全、避免重复加载
- [ ] 破坏双亲委派：TCCL（JDBC SPI）、Tomcat 隔离类加载、OSGi、热部署
- [ ] 自定义类加载器实现
- [ ] Java 9 模块化（Jigsaw）- 模块描述符、服务加载

### 4.4 垃圾回收（GC）
- [ ] 对象存活判断：引用计数法 / 可达性分析（GC Roots）
- [ ] GC Roots 包括哪些
- [ ] 强引用 / 软引用 / 弱引用 / 虚引用 + `ReferenceQueue`
- [ ] 回收算法对比：标记-清除 / 标记-复制 / 标记-整理 / 分代收集
- [ ] 新生代 GC（Minor GC）触发与晋升
- [ ] Full GC 触发条件（8 种）
- [ ] Stop The World（STW）与安全点（Safe Point）/ 安全区域（Safe Region）
- [ ] **Serial** / **Serial Old**
- [ ] **ParNew**
- [ ] **Parallel Scavenge** / **Parallel Old**（吞吐量优先）
- [ ] **CMS**：初始标记 → 并发标记 → 重新标记 → 并发清除 + 优缺点
- [ ] **G1**：Region 分区、停顿预测、Remembered Set、Card Table、Young GC / Mixed GC
- [ ] **ZGC**：染色指针、读屏障、并发整理、亚毫秒停顿
- [ ] **Shenandoah**：多阶段并发 Evacuation
- [ ] **Epsilon**：无操作 GC（性能测试）
- [ ] 选择合适的 GC 收集器

### 4.5 GC 日志与调优
- [ ] GC 日志参数（JDK 8 `-XX:+PrintGCDetails` / JDK 17+ `-Xlog:gc*`）
- [ ] GC 日志分析工具：GCeasy / GCViewer
- [ ] JVM 内存参数：`-Xms` / `-Xmx` / `-Xmn` / `-XX:MetaspaceSize` / `-XX:MaxMetaspaceSize`
- [ ] 比例参数：`-XX:SurvivorRatio` / `-XX:NewRatio` / `-XX:MaxTenuringThreshold`
- [ ] 大对象直接进入老年代：`-XX:PretenureSizeThreshold`
- [ ] G1 专用参数：`-XX:MaxGCPauseMillis` / `-XX:G1HeapRegionSize`
- [ ] 调优目标设定：停顿时间 vs 吞吐量
- [ ] OOM 排查流程：堆 / 栈 / 方法区 / 直接内存
- [ ] 内存泄漏排查：`jmap` + MAT（GC Root 引用链分析 / 大对象）
- [ ] CPU 飙升排查：`top -H` + `jstack`
- [ ] 死锁排查：`jstack` / `jconsole`
- [ ] 频繁 Full GC 优化

### 4.6 JDK 内置工具
- [ ] `jps` - Java 进程查看
- [ ] `jstat` - GC 与类加载统计
- [ ] `jinfo` - 参数查看与修改
- [ ] `jmap` - 堆转储
- [ ] `jhat` - 堆分析
- [ ] `jstack` - 线程栈
- [ ] `jcmd` - 综合诊断
- [ ] `jconsole` / `jvisualvm` / `jmc` - 可视化监控
- [ ] **Arthas**：dashboard / thread / stack / trace / redefine / ognl / vmtool
- [ ] **MAT**（Memory Analyzer Tool）：支配树 / 泄漏嫌疑人报告 / OQL

### 4.7 JIT 编译
- [ ] 解释执行 vs 编译执行
- [ ] 热点代码检测（Hot Spot Detection / 方法计数器 / 回边计数器）
- [ ] C1（Client） vs C2（Server） vs Graal JIT
- [ ] 分层编译（Tiered Compilation）
- [ ] 方法内联（Inlining）
- [ ] 逃逸分析（Escape Analysis）- 栈上分配 / 标量替换 / 锁消除
- [ ] 循环优化（循环展开 / 循环剥离 / 循环倒置）
- [ ] Code Cache 管理
- [ ] AOT（提前编译）：GraalVM Native Image

---

## 五、Spring 框架

### 5.1 IoC 容器
- [ ] IoC（控制反转）与 DI（依赖注入）设计思想
- [ ] `BeanFactory` vs `ApplicationContext`
- [ ] XML 配置、注解配置、Java Config（`@Configuration` + `@Bean`）
- [ ] `@Component` / `@Service` / `@Repository` / `@Controller` 立体化分层
- [ ] `@Autowired` / `@Resource` / `@Inject` 注入方式对比
- [ ] 构造器注入（推荐） vs Setter 注入 vs 字段注入
- [ ] `@Qualifier` / `@Primary` / `@Order`
- [ ] Bean 作用域：singleton / prototype / request / session / application / websocket
- [ ] Bean 生命周期：实例化 → 属性赋值 → 初始化 → 使用 → 销毁
- [ ] 后置处理器：`BeanPostProcessor` / `BeanFactoryPostProcessor`
- [ ] `@PostConstruct` / `@PreDestroy` / `InitializingBean` / `DisposableBean`
- [ ] `FactoryBean` vs `BeanFactory`
- [ ] `@Import` / `@ImportResource` / `@PropertySource`
- [ ] `ImportSelector` / `ImportBeanDefinitionRegistrar`
- [ ] `Conditional` 条件装配
- [ ] `Profile` 环境切换
- [ ] `@Lazy` 懒加载
- [ ] `PropertyEditor` / `Converter` / `ConversionService` 类型转换
- [ ] `Value` / `Environment` / `@ConfigurationProperties`
- [ ] SpEL（Spring Expression Language）
- [ ] 国际化（`MessageSource` / `LocaleResolver`）
- [ ] 事件机制：`ApplicationEvent` / `ApplicationListener` / `@EventListener`

### 5.2 AOP
- [ ] AOP 核心概念：Joinpoint / Pointcut / Advice / Aspect / Weaving / Target / Proxy
- [ ] 通知类型：`@Before` / `@After` / `@AfterReturning` / `@AfterThrowing` / `@Around`
- [ ] JDK 动态代理 vs CGLIB 代理选择机制
- [ ] Pointcut 表达式 && || ! 组合
- [ ] 自定义注解 + AOP 切面编程
- [ ] AOP 应用场景：日志、权限、事务、缓存、性能监控

### 5.3 事务管理
- [ ] `PlatformTransactionManager`（DataSource / JPA / Hibernate）
- [ ] 声明式事务 `@Transactional` vs 编程式事务 `TransactionTemplate`
- [ ] 事务传播行为（7 种）：REQUIRED / SUPPORTS / MANDATORY / REQUIRES_NEW / NOT_SUPPORTED / NEVER / NESTED
- [ ] 事务隔离级别：DEFAULT / READ_UNCOMMITTED / READ_COMMITTED / REPEATABLE_READ / SERIALIZABLE
- [ ] 事务失效场景（6 种）：非 public / 同类调用 / 异常不匹配 / catch 未抛 / 传播行为配置 / 数据库引擎不支持
- [ ] 事务同步管理器 `TransactionSynchronizationManager`
- [ ] 事务挂起与恢复机制
- [ ] 事务超时与回滚规则

### 5.4 Web MVC
- [ ] Spring MVC 处理流程：`DispatcherServlet` → `HandlerMapping` → `HandlerAdapter` → Controller → ViewResolver
- [ ] `@Controller` vs `@RestController`
- [ ] `@RequestMapping` / `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping`
- [ ] `@RequestParam` / `@PathVariable` / `@RequestBody` / `@RequestHeader` / `@CookieValue` / `@ModelAttribute`
- [ ] `@ResponseBody` 与 `HttpMessageConverter`（Jackson / Gson / Fastjson 自动序列化）
- [ ] `RESTful API` 设计规范
- [ ] `@ControllerAdvice` / `@RestControllerAdvice` + `@ExceptionHandler` 全局异常处理
- [ ] `@InitBinder` 数据绑定与类型转换
- [ ] `HandlerInterceptor` 拦截器（preHandle / postHandle / afterCompletion）
- [ ] `Filter` vs `Interceptor` vs `Aspect` 对比
- [ ] CORS 跨域解决方案
- [ ] 文件上传与 `MultipartFile`
- [ ] `ViewResolver` / `View` 模板渲染（JSP / Thymeleaf / FreeMarker）
- [ ] 异步请求处理：`Callable` / `DeferredResult` / `WebAsyncTask`
- [ ] Spring WebFlux（响应式编程）：`Mono` / `Flux` / `WebClient`
- [ ] 接口参数校验 `@Valid` / `@Validated` + `BindingResult`

### 5.5 Spring 其他核心
- [ ] `@Cacheable` / `@CacheEvict` / `@CachePut` / `@Caching` 缓存抽象
- [ ] `@Scheduled` + `@EnableScheduling` 定时任务
- [ ] `@Async` + `@EnableAsync` 异步方法 + `TaskExecutor` 自定义
- [ ] `@EnableTransactionManagement` 事务注解驱动
- [ ] Spring 测试：`@ContextConfiguration` / `@RunWith(SpringRunner)` / `@SpringBootTest`
- [ ] Spring Boot 测试切片：`@WebMvcTest` / `@DataJpaTest` / `@JsonTest` / `@RestClientTest`
- [ ] Spring Session 分布式会话
- [ ] Spring Security：认证 / 授权 / OAuth2 / JWT
- [ ] Spring Data：JPA / Redis / MongoDB / Elasticsearch / Cassandra
- [ ] Spring Cloud：见第七部分
- [ ] Spring 设计模式：工厂模式（BeanFactory）、单例模式（Bean）、代理模式（AOP）、模板模式（JdbcTemplate）、观察者模式（事件机制）、策略模式（Resource）、适配器模式（HandlerAdapter）

---

## 六、Spring Boot

### 6.1 自动配置
- [ ] `@SpringBootApplication`（`@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`）
- [ ] 自动配置原理：`AutoConfigurationImportSelector` → `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
- [ ] `@ConditionalOnClass` / `@ConditionalOnMissingBean` / `@ConditionalOnProperty` / `@ConditionalOnExpression` 系列
- [ ] 自定义 Starter 开发步骤
- [ ] `spring.factories` 与 `AutoConfiguration.imports`

### 6.2 配置管理
- [ ] `application.properties` / `application.yml` / `application-{profile}.yml`
- [ ] 配置加载优先级（14 种来源）
- [ ] `@ConfigurationProperties` + `@EnableConfigurationProperties`
- [ ] `@Value` 注入配置值
- [ ] `@PropertySource` 加载外部配置文件
- [ ] Profile 多环境配置
- [ ] 随机值配置（`${random.int}` / `${random.uuid}`）
- [ ] 配置加密（Jasypt 集成）

### 6.3 内嵌容器
- [ ] Tomcat / Jetty / Undertow 切换
- [ ] 内嵌容器配置（端口 / SSL / 压缩 / 连接池）
- [ ] 替换 Servlet 容器

### 6.4 数据访问
- [ ] 数据源配置：HikariCP / Druid / Tomcat JDBC Pool
- [ ] 多数据源配置 + `@Primary` + `@Qualifier`
- [ ] JDBC Template
- [ ] Spring Data JPA（Hibernate）
- [ ] MyBatis 整合 + MyBatis-Plus
- [ ] 事务管理：`@Transactional` + 多数据源分布式事务
- [ ] 读写分离实现

### 6.5 监控与运维
- [ ] Spring Boot Actuator：`/health` / `/metrics` / `/info` / `/env` / `/beans` / `/mappings`
- [ ] 自定义 Actuator Endpoint
- [ ] Metrics + Micrometer + Prometheus + Grafana 监控体系
- [ ] Spring Boot Admin 监控面板
- [ ] 日志框架（Logback / Log4j2）配置与动态调整

### 6.6 测试
- [ ] `@SpringBootTest` 集成测试
- [ ] `@WebMvcTest` Controller 切片测试
- [ ] `@DataJpaTest` Repository 切片测试
- [ ] `@RestClientTest` REST 客户端测试
- [ ] `TestRestTemplate` / `WebTestClient`
- [ ] `@MockBean` / `@SpyBean` 打桩
- [ ] `@Sql` 测试数据准备

### 6.7 部署与高级
- [ ] 可执行 JAR（Fat JAR）打包与原理
- [ ] WAR 包传统部署
- [ ] Docker 容器化 + docker-compose
- [ ] Kubernetes 部署
- [ ] GraalVM Native Image（AOT 编译）+ Spring Boot 3.x 支持
- [ ] 虚拟线程（Virtual Threads / Project Loom）集成
- [ ] Spring Boot 2.x vs 3.x 升级要点

---

## 七、Spring Cloud 微服务

### 7.1 微服务架构基础
- [ ] 微服务架构设计原则与优缺点
- [ ] 微服务 vs 单体架构对比
- [ ] 服务拆分原则（DDD 限界上下文 / 业务能力 / 功能拆分）
- [ ] Spring Cloud 版本演进（对应 Spring Boot 版本）
- [ ] Spring Cloud Alibaba 项目体系

### 7.2 服务注册与发现
- [ ] **Nacos**：AP/CP 切换、临时/持久实例、健康检查、心跳机制、保护阈值
- [ ] **Eureka**：AP 设计、自我保护机制、Renew/Evict/Cancel
- [ ] **Consul**：CP 设计、健康检查、KV Store
- [ ] **Zookeeper**：CP 设计、ZAB 协议、临时节点
- [ ] CAP 理论在注册中心中的权衡

### 7.3 服务调用
- [ ] **OpenFeign**：声明式 HTTP 客户端、动态代理、@FeignClient
- [ ] Feign 拦截器（`RequestInterceptor`）
- [ ] Feign 日志配置
- [ ] Feign 编解码器与契约
- [ ] `RestTemplate` + `@LoadBalanced`
- [ ] `WebClient` 响应式调用

### 7.4 负载均衡
- [ ] **Spring Cloud LoadBalancer**（取代 Ribbon）
- [ ] 负载均衡算法：轮询 / 加权 / 随机 / 最小连接 / 一致性哈希 / 最小活跃数
- [ ] 自定义负载均衡策略
- [ ] 重试机制

### 7.5 服务熔断与限流
- [ ] **Sentinel** 核心功能：流控、熔断降级、热点限流、系统自适应
- [ ] Sentinel 流控模式（直接 / 关联 / 链路）与效果（快速失败 / Warm Up / 排队等待）
- [ ] Sentinel 熔断策略（慢调用比例 / 异常比例 / 异常数）
- [ ] Sentinel 规则持久化（Push 模式 + Nacos）
- [ ] Sentinel @SentinelResource 注解
- [ ] **Hystrix**（已停更）：线程池隔离 vs 信号量隔离
- [ ] **Resilience4j**：模块化熔断器、限流器、重试、舱壁
- [ ] Sentinel vs Hystrix vs Resilience4j 对比

### 7.6 网关
- [ ] **Spring Cloud Gateway** vs Zuul 1.x
- [ ] Gateway 核心组件：Route / Predicate / Filter
- [ ] 内置谓词工厂（Path / Method / Header / Query / Cookie / RemoteAddr / Weight / After / Before / Between）
- [ ] 全局过滤器（GlobalFilter）与局部过滤器（GatewayFilter）
- [ ] 自定义过滤器工厂
- [ ] 网关限流（RequestRateLimiter + Redis）
- [ ] 网关跨域配置
- [ ] 网关统一鉴权
- [ ] 网关灰度发布

### 7.7 配置中心
- [ ] **Nacos Config**：长轮询监听、配置动态刷新、配置版本管理
- [ ] **Apollo**：分布式配置中心、多环境管理、灰度发布、权限控制
- [ ] `@RefreshScope` 动态刷新原理
- [ ] 配置优先级：bootstrap.yml → application.yml → Nacos → 本地
- [ ] 配置中心 vs 传统配置方案

### 7.8 分布式事务
- [ ] **Seata** AT 模式：全局事务 / 分支事务 / 全局锁 / 反向 SQL
- [ ] **Seata** TCC 模式：Try-Confirm-Cancel 手动补偿
- [ ] **Seata** Saga 模式：状态机引擎、长事务补偿
- [ ] **Seata** XA 模式：数据库原生支持
- [ ] AT vs TCC vs Saga vs XA 场景选择
- [ ] 消息最终一致性（RocketMQ 事务消息）
- [ ] 本地消息表方案

### 7.9 链路追踪
- [ ] **Spring Cloud Sleuth**（升级为 Micrometer Tracing）：Trace / Span / 采样率
- [ ] **Zipkin** 链路数据收集与展示
- [ ] **Skywalking**：无侵入探针、性能剖析、告警
- [ ] OpenTelemetry 统一标准
- [ ] 日志链路 ID 传递（MDC）

### 7.10 服务网格（Service Mesh）
- [ ] Istio / Envoy 架构
- [ ] Sidecar 代理模式
- [ ] K8s + Istio 服务治理
- [ ] Spring Cloud 与 Istio 共存策略

### 7.11 微服务其他
- [ ] 分布式 ID 方案：雪花算法 / 美团 Leaf / 百度 UidGenerator / 号段模式
- [ ] 分布式锁：Redis / Zookeeper / 数据库乐观锁
- [ ] 接口幂等性设计（唯一键 / 状态机 / Token）
- [ ] 分布式 Session：Spring Session + Redis
- [ ] OAuth2 / JWT 认证授权体系
- [ ] 灰度发布（金丝雀发布 / 蓝绿部署 / 滚动更新）
- [ ] 日志收集（ELK / EFK / Loki）
- [ ] 容器化部署（Docker + K8s）

---

## 八、关系型数据库（MySQL）

### 8.1 存储引擎
- [ ] **InnoDB**：事务、行锁、外键、崩溃恢复、MVCC
- [ ] **MyISAM**：表锁、全文索引、压缩表、不支持事务
- [ ] **Memory**：哈希索引、内存表、重启数据丢失
- [ ] 引擎选择场景分析
- [ ] InnoDB 逻辑存储结构：表空间（Tablespace）→ 段（Segment）→ 区（Extent）→ 页（Page）→ 行（Row）

### 8.2 索引
- [ ] B+Tree 数据结构与查找过程
- [ ] B+Tree vs B-Tree vs 红黑树 vs 哈希索引
- [ ] 聚簇索引（主键索引） vs 非聚簇索引（二级索引）
- [ ] 联合索引（复合索引）与最左前缀原则
- [ ] 索引下推（ICP）
- [ ] 覆盖索引与回表查询
- [ ] 索引失效场景（7+ 种）
- [ ] 索引创建原则（高区分度 / 查询条件 / ORDER BY / GROUP BY）
- [ ] 索引优化：冗余索引检查、不可见索引（MySQL 8.0）

### 8.3 SQL 优化
- [ ] `EXPLAIN` 执行计划详解：id / select_type / table / type / possible_keys / key / key_len / ref / rows / Extra
- [ ] type 字段详解：system → const → eq_ref → ref → range → index → ALL
- [ ] Extra 常见值：Using index / Using where / Using index condition / Using filesort / Using temporary
- [ ] 分页优化：子查询延迟关联 / 游标分页（WHERE id > ? LIMIT ?）
- [ ] JOIN 优化：小表驱动大表、Join Buffer、索引关联字段
- [ ] 子查询优化（JOIN 替代 / EXISTS 替代 IN）
- [ ] `COUNT` 优化：COUNT(*) vs COUNT(1) vs COUNT(列)
- [ ] `GROUP BY` / `ORDER BY` 优化：索引排序 / 文件排序
- [ ] `OR` 条件优化（UNION ALL 替代）
- [ ] 大表 DDL：pt-online-schema-change / gh-ost

### 8.4 事务与锁
- [ ] ACID 特性实现原理（A-undo log, C-redo log+undolog, I-锁, D-redo log）
- [ ] 事务隔离级别：READ UNCOMMITTED / READ COMMITTED / REPEATABLE READ（默认）/ SERIALIZABLE
- [ ] 脏读 / 不可重复读 / 幻读
- [ ] **MVCC**：ReadView、undolog 版本链、快照读 vs 当前读
- [ ] **Next-Key Lock**（Record Lock + Gap Lock）- 间隙锁解决幻读
- [ ] 行锁 / 表锁 / 意向锁（IS / IX）
- [ ] 死锁产生条件与检测机制
- [ ] 死锁排查：`SHOW ENGINE INNODB STATUS`
- [ ] 大事务危害与避免方法
- [ ] 锁等待超时参数（`innodb_lock_wait_timeout`）

### 8.5 日志
- [ ] **Redo Log**：WAL 机制、崩溃恢复、物理日志、循环写
- [ ] **Undo Log**：事务回滚、MVCC 快照、逻辑日志
- [ ] **Binlog**（二进制日志）：ROW / STATEMENT / MIXED 格式、主从复制、数据恢复
- [ ] Redo Log vs Binlog 区别（物理 vs 逻辑、InnoDB vs Server 层）
- [ ] 两阶段提交（2PC）保证 Redo Log + Binlog 一致性

### 8.6 主从复制与高可用
- [ ] 主从复制原理：Binary Log → Relay Log → SQL Thread
- [ ] 异步复制 / 半同步复制 / 全同步复制（MGR）
- [ ] 并行复制（Slave Parallel Workers）
- [ ] 主从延迟原因与解决方案
- [ ] 读写分离：强制走主库 / 延迟容忍
- [ ] 高可用方案：MHA / MGR / InnoDB Cluster / ProxySQL

### 8.7 分库分表
- [ ] 分库分表策略：范围分片 / 哈希分片 / 一致性哈希（虚拟节点）
- [ ] 分库分表后事务问题（分布式事务）
- [ ] 跨节点查询：跨库 JOIN / 全局聚合 / 排序 / 分页
- [ ] 全局主键 ID 方案
- [ ] 分库分表中间件：**ShardingSphere-JDBC** / **MyCat**
- [ ] 分库分表 vs NoSQL 选择

### 8.8 数据库设计
- [ ] 三大范式（1NF / 2NF / 3NF）与反范式设计
- [ ] ER 模型设计
- [ ] 字段类型选择：数值/字符串/日期/JSON/BLOB/TEXT
- [ ] 主键设计（自增 ID / UUID / 雪花 ID / 雪花算法变种）
- [ ] 表设计原则：字段长度合理、冗余控制、适当反范式
- [ ] 索引设计规范

### 8.9 备份与恢复
- [ ] 逻辑备份（mysqldump）
- [ ] 物理备份（Xtrabackup）
- [ ] 增量备份与全量备份策略
- [ ] 误删数据恢复（binlog + 备份恢复）

---

## 九、Redis

### 9.1 数据结构与底层实现
- [ ] **String** - SDS（简单动态字符串）、int / embstr / raw 编码
- [ ] **List** - 3.2 前 ziplist + linkedlist → 3.2+ quicklist（双向链表 + 压缩列表）
- [ ] **Hash** - ziplist → hashtable（元素少时压缩列表优化）
- [ ] **Set** - intset → hashtable
- [ ] **ZSet** - ziplist → skiplist（跳表）+ hashtable
- [ ] **BitMap** - SETBIT / GETBIT / BITCOUNT / BITOP 位运算
- [ ] **HyperLogLog** - 基数统计（PFADD / PFCOUNT / PFMERGE）
- [ ] **Geo** - GeoHash 编码（GEOADD / GEORADIUS / GEODIST）
- [ ] **Stream** - 消息队列（XADD / XREAD / XGROUP / XACK）

### 9.2 核心功能
- [ ] 过期策略：惰性删除 + 定期删除
- [ ] 内存淘汰策略（8 种）
- [ ] 持久化：RDB（`SAVE` / `BGSAVE`） vs AOF（`appendfsync always`/`everysec`/`no`）
- [ ] 混合持久化（Redis 4.0+）
- [ ] AOF 重写（`BGREWRITEAOF`）
- [ ] 事务：MULTI / EXEC / DISCARD / WATCH（CAS 乐观锁）
- [ ] Pipeline 批量请求
- [ ] 发布订阅（PUB / SUB） vs Stream 消息队列
- [ ] Lua 脚本：EVAL / EVALSHA / SCRIPT LOAD
- [ ] Redis 6.0 多线程 IO 模型
- [ ] Redis 7.0 新特性

### 9.3 缓存设计
- [ ] 缓存穿透（布隆过滤器 / 空值缓存）
- [ ] 缓存击穿（互斥锁 / 热点数据永不过期）
- [ ] 缓存雪崩（随机过期时间 / 多级缓存 / 集群高可用 / 限流降级）
- [ ] 缓存与数据库一致性方案
  - [ ] 先更新 DB 后删缓存（延迟双删）
  - [ ] 消息队列异步同步
  - [ ] Canal 监听 binlog
- [ ] 热 Key / 大 Key 发现与处理（拆分 / 本地缓存 / 限流）
- [ ] 多级缓存架构：本地缓存（Caffeine） + Redis + DB

### 9.4 分布式锁
- [ ] 单机 Redis 锁：`SET key value NX EX seconds`
- [ ] Lua 脚本原子解锁
- [ ] Redisson：看门狗（Watch Dog）自动续期
- [ ] RedLock 算法与争议
- [ ] RedLock vs ZK 分布式锁对比

### 9.5 集群与高可用
- [ ] **主从复制**：全量复制 + 增量复制 + psync2
- [ ] **Sentinel（哨兵）**：监控 / 选举（Raft）/ 故障转移 / 脑裂问题 / quorum
- [ ] **Cluster（集群）**：16384 哈希槽 / Gossip 协议 / MOVED 重定向 / ASK 重定向
- [ ] Cluster 扩容与 rehash
- [ ] Redis 部署模式选择

### 9.6 深度话题
- [ ] 单线程模型为什么快：内存 / IO 多路复用（epoll）/ 避免上下文切换
- [ ] IO 多路复用：select / poll / epoll 区别
- [ ] 内存碎片与优化（`jemalloc` / `activedefrag`）
- [ ] 异步删除：UNLINK / FLUSHALL ASYNC / FLUSHDB ASYNC
- [ ] Redis 键设计规范（前缀命名 / 短 Key / 内存估算）
- [ ] Redis 变慢排查（慢查询 / BigKey / Fork / AOF 写 / 内存碎片 / 网络带宽）

---

## 十、消息队列（RocketMQ）

### 10.1 消息队列基础
- [ ] 消息队列应用场景：异步解耦 / 削峰填谷 / 日志收集 / 最终一致性
- [ ] MQ 选型对比：RocketMQ vs Kafka vs RabbitMQ vs Pulsar vs ActiveMQ
- [ ] 消息队列核心概念：Producer / Consumer / Broker / Topic / Queue

### 10.2 RocketMQ 核心架构
- [ ] **NameServer**：无状态路由中心、心跳检测、路由表
- [ ] **Broker**：主从架构、消息存储、同步复制/异步复制
- [ ] **Producer**：同步发送 / 异步发送 / 单向发送
- [ ] **Consumer**：Push 模式 vs Pull 模式（长轮询）
- [ ] **Topic** 与 **Queue** 的关系
- [ ] 消费模式：集群消费 / 广播消费

### 10.3 消息存储
- [ ] 存储结构：**CommitLog**（顺序写）+ **ConsumeQueue**（逻辑队列）+ **IndexFile**（索引）
- [ ] 零拷贝技术：`MappedByteBuffer` + `FileChannel.transferTo` + `sendfile`
- [ ] 刷盘机制：同步刷盘 vs 异步刷盘
- [ ] 过期文件删除机制

### 10.4 消息特性
- [ ] **顺序消息**：全局顺序 vs 分区顺序
- [ ] **事务消息**：半消息 → 本地事务 → Commit/Rollback → 回查
- [ ] **延时消息**：18 个延迟级别
- [ ] **定时消息**（5.0+）
- [ ] **批量消息**：批量发送 / 批量消费
- [ ] **消息过滤**：Tag 过滤 / SQL92 过滤
- [ ] **消息重试**：重试次数 / 重试间隔
- [ ] **死信队列（DLQ）**：产生原因与处理流程
- [ ] **消息轨迹**：消息全链路追踪

### 10.5 消息可靠性
- [ ] 消息不丢失三端保障
  - [ ] 生产端：同步发送 + 重试 + 回调确认
  - [ ] Broker 端：同步刷盘 + 同步复制
  - [ ] 消费端：业务处理完再 ACK
- [ ] 消息幂等消费：唯一键去重 / 状态机 / 事务表
- [ ] 消息补偿机制

### 10.6 消费负载均衡
- [ ] 集群消费负载均衡：平均分配 / 循环分配 / 一致性哈希
- [ ] 广播消费：每个消费者消费所有队列
- [ ] Rebalance 触发条件与影响

### 10.7 运维与调优
- [ ] Broker 参数调优：`maxMessageSize` / `fileReservedTime` / `transientStorePoolEnable`
- [ ] 消息堆积处理：消费者扩容 / 重置消费位点 / 限流方案
- [ ] 回溯消费：按时间重置消费位点
- [ ] RocketMQ Dashboard 监控
- [ ] 双主双从部署架构
- [ ] RocketMQ 5.x 新特性

---

## 十一、MinIO 对象存储

### 11.1 基础概念
- [ ] 对象存储 vs 文件存储 vs 块存储
- [ ] S3 兼容 API
- [ ] MinIO vs FastDFS vs HDFS vs Ceph vs 云 OSS
- [ ] 核心概念：Bucket / Object / Access Key / Secret Key

### 11.2 架构与高可用
- [ ] **纠删码（Erasure Coding）**：数据块 + 校验块、N+M 冗余
- [ ] **Bit Rot Protection**：哈希校验防止位衰减
- [ ] 分布式 MinIO 部署要求（4 节点起）
- [ ] 数据加密：SSE-C / SSE-S3 / SSE-KMS
- [ ] **版本控制**：历史版本管理
- [ ] **对象锁定（WORM）**：写一次读多次

### 11.3 功能与使用
- [ ] 分片上传（Multipart Upload）
- [ ] 预签名 URL（Presigned URL）临时分享
- [ ] 事件通知：Webhook / Kafka / AMQP / Redis / PostgreSQL / Elasticsearch
- [ ] 生命周期管理：自动过期 / 迁移
- [ ] 桶复制（Bucket Replication）
- [ ] 静态网站托管
- [ ] IAM 权限控制（用户 / 组 / 策略）

### 11.4 Java 集成
- [ ] MinIO Java SDK：`MinioClient` / `putObject` / `getObject` / `removeObject` / `listObjects`
- [ ] Spring Boot 整合 MinIO
- [ ] 大文件分片上传后端实现
- [ ] 图片处理 + MinIO 结合

---

## 十二、定时任务（XXL-Job）

### 12.1 架构原理
- [ ] 调度中心（Admin） vs 执行器（Executor）
- [ ] 调度中心集群：数据库行锁（`for update`）保证调度一致性
- [ ] 执行器注册与发现机制
- [ ] HTTP 回调通信模式
- [ ] 心跳检测与健康检查

### 12.2 任务类型
- [ ] Bean 模式：`@XxlJob` 注解 / 继承 `IJobHandler`
- [ ] GLUE 模式：在线编辑 / 动态编译 / 热加载
- [ ] 脚本任务：Shell / Python / PHP / Node.js / PowerShell

### 12.3 路由策略与分片
- [ ] 10 种路由策略详解：First / Last / Round / Random / Consistent Hash / LFU / LRU / Failover / Busy Over / Sharding Broadcast
- [ ] **分片广播**：`shardIndex` + `shardTotal`
- [ ] 分片任务设计：数据按 ID 取模 / 按时间范围
- [ ] 动态分片与扩容缩容

### 12.4 任务调度与生命周期
- [ ] 调度策略：Cron / 固定速度 / 固定延迟 / 时间线调度
- [ ] 阻塞处理策略：串行 / 丢弃后续 / 覆盖之前 / 单机串行
- [ ] 任务超时控制
- [ ] 失败重试机制
- [ ] 任务依赖（DAG 有向无环图 / 父子任务）
- [ ] 调度日志与执行日志

### 12.5 运维
- [ ] 报警机制：邮件 / 企业微信 / 钉钉 / 飞书 / 自定义
- [ ] 动态暂停 / 启动 / 删除任务
- [ ] 调度过期策略
- [ ] XXL-Job vs Quartz 对比
- [ ] Docker + K8s 容器化部署

---

## 十三、开发工具与工程化

### 13.1 版本控制
- [ ] Git 工作流：GitFlow / GitHub Flow / GitLab Flow
- [ ] 分支管理策略
- [ ] `rebase` vs `merge`
- [ ] Cherry-pick / Stash / Reset / Revert
- [ ] 代码审查（Code Review）最佳实践

### 13.2 构建工具
- [ ] **Maven**：POM / 坐标 / 依赖传递 / 冲突解决 / 生命周期 / 插件体系
- [ ] **Gradle**：Groovy/Kotlin DSL / 增量构建 / 依赖缓存
- [ ] Maven vs Gradle 对比
- [ ] 私有仓库：Nexus / Artifactory

### 13.3 持续集成/部署（CI/CD）
- [ ] **Jenkins**：Pipeline / 多分支构建 / 分布式构建
- [ ] **GitLab CI**：`.gitlab-ci.yml` / Runner / 缓存
- [ ] **GitHub Actions**：Workflow / Job / Step
- [ ] 容器化构建：Docker Image 打包与推送
- [ ] 自动化部署：Shell / Ansible / Helm

### 13.4 容器与编排
- [ ] **Docker**：镜像 / 容器 / Dockerfile / Docker Compose
- [ ] **Kubernetes**：Pod / Deployment / Service / ConfigMap / Secret / Ingress
- [ ] K8s 弹性伸缩（HPA / VPA）
- [ ] Helm 包管理
- [ ] 服务网格（Istio / Linkerd）

### 13.5 监控与运维
- [ ] **Prometheus** + **Grafana** 监控体系
- [ ] **ELK**（Elasticsearch + Logstash + Kibana）日志
- [ ] **Skywalking** / **Pinpoint** APM
- [ ] **Arthas** 在线诊断
- [ ] **Jenkins** + **SonarQube** 代码质量

### 13.6 压力测试
- [ ] JMeter 压测脚本编写
- [ ] 性能指标：TPS / QPS / RT（响应时间）/ 吞吐量
- [ ] 全链路压测方案
- [ ] 慢 SQL / 慢接口识别与优化

---

## 十四、系统设计与架构

### 14.1 设计原则
- [ ] SOLID 原则（单一职责 / 开闭 / 里氏替换 / 接口隔离 / 依赖倒置）
- [ ] KISS / YAGNI / DRY
- [ ] 组合优于继承
- [ ] 面向接口编程
- [ ] 控制反转与依赖注入

### 14.2 设计模式
- [ ] **创建型**：单例 / 工厂 / 抽象工厂 / 建造者 / 原型
- [ ] **结构型**：代理 / 适配器 / 装饰器 / 外观 / 桥接 / 组合 / 享元
- [ ] **行为型**：策略 / 模板方法 / 观察者 / 责任链 / 迭代器 / 命令 / 状态 / 访问者 / 中介者 / 备忘录 / 解释器
- [ ] **并发型**：Future / 读写锁 / 线程池

### 14.3 架构模式
- [ ] 分层架构（Controller → Service → DAO）
- [ ] 微服务架构（Spring Cloud）
- [ ] 事件驱动架构（EDA）
- [ ] CQRS（命令查询职责分离）
- [ ] 事件溯源（Event Sourcing）
- [ ] 六边形架构（端口适配器）
- [ ] 整洁架构（Clean Architecture）

### 14.4 分布式理论
- [ ] **CAP** 理论：一致性 / 可用性 / 分区容错性
- [ ] **BASE** 理论：基本可用 / 软状态 / 最终一致性
- [ ] **一致性模型**：强一致性 / 弱一致性 / 最终一致性 / 因果一致性 / 单调读
- [ ] **Paxos** 算法
- [ ] **Raft** 算法：Leader 选举 / 日志复制 / 安全性
- [ ] **共识算法**应用：ETCD / ZooKeeper / Nacos CP
- [ ] **分布式事务**：2PC / 3PC / TCC / Saga

### 14.5 高可用架构
- [ ] 冗余设计：多副本 / 主从 / 多活
- [ ] 限流策略：令牌桶 / 漏桶 / 滑动窗口
- [ ] 降级与熔断：接口降级 / 熔断恢复
- [ ] 超时与重试：连接超时 / 读取超时 / 指数退避
- [ ] 负载均衡：四层 / 七层 / 一致性哈希
- [ ] 灾备：同城双活 / 两地三中心

---

## 十五、常用技术总结

| 类别 | 技术 | 掌握程度 |
|---|---|---|
| 基础 | Java 8~21 核心 | 精通 |
| 框架 | Spring / Spring Boot / Spring Cloud | 精通 |
| ORM | MyBatis / MyBatis-Plus / JPA | 精通 |
| 数据库 | MySQL（InnoDB / 索引 / 事务 / 锁 / 优化） | 精通 |
| 缓存 | Redis（数据结构 / 集群 / 分布式锁 / 缓存设计） | 精通 |
| 消息队列 | RocketMQ / Kafka（选学） | 深入 |
| 对象存储 | MinIO / OSS | 熟练 |
| 定时任务 | XXL-Job | 熟练 |
| 注册配置 | Nacos / Apollo | 深入 |
| 网关 | Spring Cloud Gateway | 深入 |
| 熔断限流 | Sentinel | 深入 |
| 搜索 | Elasticsearch | 熟练 |
| 链路追踪 | Skywalking / Zipkin | 熟练 |
| 容器 | Docker / Kubernetes | 熟练 |
| CI/CD | Jenkins / GitLab CI / GitHub Actions | 熟练 |
| 监控 | Prometheus + Grafana | 了解 |
| 压测 | JMeter | 了解 |
| 项目管理 | Maven / Gradle | 精通 |
| 版本控制 | Git | 精通 |
| 设计 | 设计模式 / 架构模式 / 分布式理论 | 深入 |

---

> **说明**：本大纲按模块组织，每个知识点前的 `[ ]` 可用于学习进度追踪。建议按照从基础到进阶的顺序逐个模块攻克，配合面试题文件进行查漏补缺。
