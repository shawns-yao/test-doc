# Spring 框架

## 一、核心原理

# 1.1 Spring IoC（控制反转）的原理是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心含义：</b>IoC（控制反转）把对象的创建、依赖组装和生命周期管理从业务代码交给 Spring 容器；业务类只声明“需要什么”，不再负责 <code>new</code> 具体实现。依赖方向由“业务类主动找依赖”变为“容器注入依赖”。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">容器流程：</b>启动时读取注解、配置类或 XML，解析为 <code>BeanDefinition</code>；根据定义实例化 Bean，解析构造器/属性依赖，执行后置处理器和初始化回调，最后按作用域缓存并提供给调用方。<code>ApplicationContext</code> 是常用的完整容器，底层能力来自 <code>BeanFactory</code>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">注入方式：</b>构造器注入最适合必需依赖和不可变对象，也便于单元测试；Setter 注入适合可选依赖；字段注入写法简单但隐藏依赖、测试不便，生产代码通常优先构造器注入。多个候选 Bean 要用 <code>@Qualifier</code> 或 <code>@Primary</code> 消歧。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">价值与边界：</b>IoC 降低模块耦合，便于替换实现、统一配置和测试，但容器启动、代理和反射会增加理解成本；手动 <code>new</code> 出来的对象不受容器管理，也不会自动获得注入、事务或 AOP 能力。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>循环依赖如何处理（Setter/字段注入可借助三级缓存，构造器循环依赖通常无法解决）；Bean 默认作用域是什么（singleton）；为什么推荐构造器注入（依赖显式、对象可不变、失败更早）。</div>
  </div>
</div>

---

# 1.2 Spring AOP（面向切面编程）的原理是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心思想：</b>AOP 把日志、事务、权限、监控等横切逻辑从业务代码中抽离，在不修改核心业务的情况下统一织入。Spring AOP 主要拦截 Spring Bean 的方法执行，适合处理重复、与主业务相对独立的逻辑。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">关键概念：</b><code>JoinPoint</code> 是可拦截位置；<code>Pointcut</code> 决定匹配哪些方法；<code>Advice</code> 是增强逻辑，包括 Before、After、AfterReturning、AfterThrowing 和 Around。多个切面会形成拦截器链，按优先级执行。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">代理方式：</b>目标类实现接口时通常使用 JDK 动态代理；没有接口时可使用 CGLIB 生成子类代理。调用链是“外部调用 → 代理对象 → 拦截器链 → 目标方法 → 返回/异常处理”，<code>@Around</code> 通过 <code>proceed()</code> 决定是否继续执行。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">典型应用：</b><code>@Transactional</code> 由事务拦截器完成开启、提交和回滚；日志切面记录 traceId、耗时和异常；权限切面在入口校验身份与资源；审计切面记录敏感操作。切面应保持幂等、低延迟，并避免吞掉业务异常。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">常见失效点：</b>同类内部调用没有经过代理，注解可能不生效；对象由 <code>new</code> 创建、不在 Spring 容器中也不会被织入；<code>private</code>、<code>final</code> 方法和自调用场景要结合代理类型判断。需要时拆分 Bean 或改用编程式方案。</div>
  </div>
</div>

---

# 1.3 Spring 循环依赖如何解决（三级缓存）？为什么三级不是两级？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：招银、腾讯云智、拼多多</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>循环依赖 = A 依赖 B、B 又依赖 A。Spring 用<b>三级缓存</b>解决<b>单例 + 构造器注入之外</b>的循环依赖：① 一级：成品单例池（singletonObjects）；② 二级：早期单例（earlySingletonObjects，已实例化未完全初始化）；③ 三级：单例工厂（singletonFactories，存 ObjectFactory 用于生成早期引用）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">解决流程（A → B → A）：</b>① 创建 A：实例化（new）→ 放入<b>三级缓存</b>（工厂）→ 填充属性发现需要 B；② 创建 B：实例化 → 三级缓存 → 填充属性发现需要 A；③ B 从<b>三级缓存</b>取 A 的工厂生成<b>早期引用</b>（提前暴露，此时 A 未完成属性填充）→ 放入二级缓存 → B 完成初始化；④ A 拿到 B，完成自己的属性填充和初始化 → 移入一级缓存，三级缓存删除。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">为什么需要三级而不是两级：</b>二级缓存也能解决循环依赖，但<b>三级缓存为了支持 AOP 代理</b>——若 A 需要代理，早期引用必须用代理对象，而不是原始对象。三级缓存放的是 ObjectFactory，在「有人真正需要早期引用」时才调用工厂生成（可在此织入代理）；如果只有两级，所有 bean 在实例化后都要立即创建代理，浪费且无法区分是否需要代理。两级缓存 + 提前代理可以工作，但三级缓存的懒加载设计更优。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>构造器注入的循环依赖<b>无法解决</b>（实例化就需要对方，提前暴露来不及）——报 BeanCurrentlyInCreationException；prototype 作用域不缓存也无法解决；循环依赖本质是设计坏味道，能避免就避免（拆依赖/延迟注入 @Lazy）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么构造器注入循环依赖解决不了（实例化阶段就卡住）；@Lazy 怎么破循环依赖（代理占位，真正调用时才初始化）；循环依赖和 AOP 的关系（三级缓存 + 提前代理）。</div>
  </div>
</div>

---

# 1.4 事务失效有哪些场景？同类内部调用为什么失效？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：熙牛医疗、领星</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>Spring 事务基于 AOP 代理：调用 <code>@Transactional</code> 方法时，实际调用的是<b>代理对象</b>，代理在方法前后开启/提交/回滚事务。任何「绕过代理」的调用都会让事务失效。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">失效场景清单：</b>① <b>同类内部调用（最常见）</b>：类内部 <code>this.methodB()</code> 调 <code>@Transactional</code> 方法——this 是原始对象不是代理，事务不生效；② <b>方法非 public</b>：Spring 默认只代理 public 方法（protected/private 不生效）；③ <b>异常被吞</b>：方法内 try-catch 捕获异常不抛出，事务无法感知回滚；④ <b>异常类型不对</b>：默认只回滚 RuntimeException/Error，checked 异常（Exception）不回滚（除非 <code>rollbackFor</code>）；⑤ <b>自建代理未生效</b>：类没有被 Spring 管理（new 出来的）、或 <code>@Transactional</code> 加在接口/类但代理配置问题；⑥ <b>传播行为配置</b>：<code>REQUIRES_NEW</code> 内层独立事务、<code>NOT_SUPPORTED</code> 挂起事务；⑦ <b>数据库不支持事务</b>：MyISAM 表；⑧ 多线程调用（新线程不在原事务上下文）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">同类内部调用怎么解决：</b>① 注入自身代理（<code>@Autowired</code> 自己，Spring 支持循环注入自身代理）或 <code>ApplicationContext.getBean()</code> 拿代理调用；② 把事务方法拆到另一个 Bean（独立类）；③ 用 <code>TransactionTemplate</code> 编程式事务（代码内包事务逻辑，不依赖代理）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么 Spring 不处理内部调用（代理基于外部调用拦截）；编程式事务和声明式事务的区别；rollbackFor=Exception.class 什么时候必须写（方法抛 checked 异常也要回滚时）。</div>
  </div>
</div>

---

# 1.5 为什么用 BeanPostProcessor？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>BeanPostProcessor（BPP）是 Spring 在 Bean 初始化前后提供的扩展点。容器创建并完成属性填充后，会依次调用 <code>postProcessBeforeInitialization</code> 和 <code>postProcessAfterInitialization</code>，允许对 Bean 做统一检查、包装或替换。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">常见用途：</b>前置阶段可做默认值填充、校验和标记；后置阶段可返回代理对象，实现 AOP、事务、异步、缓存和监控等能力。<code>AutowiredAnnotationBeanPostProcessor</code> 负责处理注入注解，自动代理创建器则在后置阶段判断是否需要生成代理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">执行位置：</b>BPP 只覆盖容器管理的 Bean，且发生在初始化回调附近；它不是 Bean 生命周期全部步骤的替代品。实现类本身会被容器优先创建，若依赖其他 Bean，要注意实例化顺序和提前暴露代理的问题。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">使用风险：</b>后置处理器可以返回不同对象，可能导致类型判断、循环依赖和调试困难；处理器逻辑应尽量轻量、幂等，避免在其中执行远程调用。需要修改 Bean 定义时应区分 <code>BeanFactoryPostProcessor</code>，它处理的是 BeanDefinition 而不是实例。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>BPP 与 BeanFactoryPostProcessor 的区别（实例后置处理 vs BeanDefinition 后置处理）；AOP 代理在哪儿生成（通常在 BPP 后置阶段）；为什么 <code>new</code> 出来的对象不生效（没有经过容器生命周期）。</div>
  </div>
</div>

---

# 1.6 ApplicationContext 和 BeanFactory 的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">定位：</b>BeanFactory 是最底层的 IOC 容器接口，核心是「注册、获取、管理 Bean」；ApplicationContext 是在 BeanFactory 之上的<b>完整企业容器</b>，功能更全。</div>
    <div style="display:grid;grid-template-columns:150px minmax(0,1fr);column-gap:12px;row-gap:6px;margin:8px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">BeanFactory</div>
      <div>依赖注入、Bean 生命周期管理、基础作用域支持。</div>
      <div style="color:#3A5FBF;font-weight:600;">ApplicationContext</div>
      <div>额外提供：国际化（MessageSource）、事件发布/监听（ApplicationEventPublisher）、资源加载（ResourceLoader）、环境与配置体系（Environment）、自动注册 BeanPostProcessor（更好集成 AOP、事务）。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">加载时机：</b>BeanFactory 偏<b>延迟加载</b>（getBean 时才创建）；ApplicationContext 默认<b>容器启动时预实例化</b>单例 Bean——启动更「重」，但错误能更早暴露。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">生产选型：</b>现代 Spring 应用要用到事件、配置、AOP、事务、自动装配等能力，ApplicationContext 开箱即用，开发和治理成本更低。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>BeanFactory 是「能用的最小容器」，ApplicationContext 是「企业级完整容器」；实际项目默认选 ApplicationContext。</div>
</div>

---

# 1.7 Spring AOP 有哪些常见场景？核心概念如何理解？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">常见场景：</b>① <b>日志与链路追踪</b>——统一记录接口入参、耗时、traceId，统计方法 RT、成功率、异常率上报监控系统，不污染业务代码；② <b>事务管理</b>——<code>@Transactional</code> 本质就是 AOP 在方法前后做事务开启/提交/回滚；③ <b>权限与鉴权</b>——在 Controller/Service 入口做统一权限校验，失败直接拦截；④ <b>审计与合规</b>——对敏感操作统一留痕（谁在什么时候做了什么）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">实现原理：</b>Spring 给目标对象「套代理」，调用先进入代理，再执行切面逻辑，最后调用目标方法。三个关键概念：<code>JoinPoint</code>（可被拦截的位置，Spring 里主要是方法执行）、<code>Pointcut</code>（匹配哪些方法）、<code>Advice</code>（增强逻辑：Before/After/Around/AfterThrowing）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">代理方式：</b>有接口默认用 <b>JDK 动态代理</b>（基于接口）；无接口用 <b>CGLIB</b> 生成子类代理（Spring Boot 里很多场景走 CGLIB）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">执行链路（最常考）：</b>外部调用 → 代理对象 → 拦截器链（多个 Advice）→ 目标方法 → 返回/异常处理。<code>@Around</code> 可以决定是否继续执行目标方法（<code>proceed()</code>）。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>Spring AOP 是「代理 + 拦截器链」机制，把日志、事务、鉴权等横切能力从业务代码里抽出来统一治理。</div>
</div>

---

# 1.8 事务传播行为常见用法？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念：</b>事务传播记成「方法 A 调方法 B 时，B 要不要共用 A 的事务」，最常用两个：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">REQUIRED（默认）</b>
        <div style="margin:4px 0 0;">有事务就加入当前事务，没事务就新开一个——A 和 B 通常「一荣俱荣、一损俱损」，任一抛异常都可能一起回滚。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">适合：</b>主业务链路（下单、扣库存、写订单）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">REQUIRES_NEW</b>
        <div style="margin:4px 0 0;">不管外层有没有事务，B 都新开事务；外层事务会被挂起，B 提交/回滚后再恢复外层——B 的提交结果与 A 解耦，A 后续失败也不影响 B 已提交。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">适合：</b>审计日志、操作留痕、补偿记录（必须落库）。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">两个常见点：</b>① 为什么「写了 <code>@Transactional</code> 还不生效」——同类内部自调用不会走代理，传播行为也不会生效；② <code>REQUIRES_NEW</code> 不能滥用——会增加事务数量、连接占用和锁竞争，只给「必须独立提交」的小操作用。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b><code>REQUIRED</code> 保证主流程原子性，<code>REQUIRES_NEW</code> 用来做与主事务解耦的独立落库；选型本质是「一致回滚」还是「独立留痕」。</div>
</div>

---

# 1.9 配置治理的最佳实践？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">配置应按<b>环境分层</b>（开发/测试/生产隔离），<b>敏感信息外置</b>到密钥系统，避免硬编码进仓库；再通过<b>配置中心</b>做版本管理和灰度发布，并明确优先级与回滚策略，减少误配事故。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">要点：</b>① 配置与代码分离（ConfigMap/环境变量，不同环境不同配置）；② 密钥不进仓库（密钥管理系统注入）；③ 版本化 + 灰度 + 回滚预案；④ 优先级明确（本地 > 环境 > 配置中心 > 默认值）。</div>
  </div>
</div>

## 二、Bean 生命周期

---

# 2.1 Spring Bean 从实例化到销毁的完整生命周期流程是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">以单例 Bean 为例，流程大致是：</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">Spring 读取 BeanDefinition，实例化 Bean。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">通过依赖注入填充属性。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">执行 <code>Aware</code> 接口回调，让 Bean 获取 BeanName、BeanFactory 等容器信息。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">执行 <code>BeanPostProcessor</code> 的前置处理。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">5</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">执行初始化方法，例如 <code>@PostConstruct</code>、<code>InitializingBean.afterPropertiesSet()</code> 和自定义 <code>init-method</code>。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">6</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">执行 <code>BeanPostProcessor</code> 的后置处理，AOP 代理通常在这一阶段生成。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">7</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">Bean 放入单例池并对外提供。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">8</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">容器关闭时执行 <code>@PreDestroy</code>、<code>DisposableBean.destroy()</code> 和自定义 <code>destroy-method</code>。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;">实际顺序要结合具体处理器和配置确认；构造器注入发生在属性填充前，原型 Bean 默认由容器创建但不负责完整销毁。</div>
  </div>
</div>

---

# 2.2 MyBatis 一级/二级缓存？`#{}` 和 `${}` 的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：浩鲸、货拉拉</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">一级缓存（默认开启）</b>
        <div style="margin:4px 0 0;"><b>SqlSession 级别</b>：同一 SqlSession 内相同 SQL 直接返回缓存；SqlSession 关闭/提交/更新时清空。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>Spring 集成时 SqlSession 默认每次操作新建（一级缓存基本不生效）；并发问题少见。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">二级缓存（需配置开启）</b>
        <div style="margin:4px 0 0;"><b>namespace（Mapper）级别</b>：跨 SqlSession 共享；多表操作要小心脏数据（另一 Mapper 更新后本缓存不失效）——<b>默认不推荐开启</b>，命中率低 + 一致性问题。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">#{} vs ${}：</b><code>#{}</code> 是<b>预编译占位符</b>（PreparedStatement 的 ?，参数安全绑定，防 SQL 注入）；<code>${}</code> 是<b>字符串拼接</b>（直接替换进 SQL，有注入风险）——<b>只有动态表名/列名/排序字段等无法参数化的场景才用 ${}，且必须白名单校验</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">字段与属性不一致：</b>① 开启驼峰映射（<code>map-underscore-to-camel-case</code>，<code>user_name → userName</code>）；② <code>resultMap</code> 显式映射；③ 别名（<code>SELECT user_name AS userName</code>）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>一级缓存什么时候失效（提交/更新/关闭 SqlSession）；为什么二级缓存可能读到脏数据（其他 namespace 更新不触发本缓存失效）；${} 用在 where 里为什么危险（拼接可注入）。</div>
  </div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
