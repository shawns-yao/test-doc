# Java

## 一、Java 集合

# 1.1 HashMap 底层数据结构是什么？如何扩容？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：快手、字节、腾讯、转转</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">底层结构：</b>JDK 8 为<b>数组 + 链表 + 红黑树</b>。key 的 <code>hashCode()</code> 经扰动（<code>h ^ (h &gt;&gt;&gt; 16)</code>）后与 <code>(n - 1)</code> 做位与定位桶；哈希冲突时用链表存储（尾插法），链表长度 &gt; 8 且数组长度 ≥ 64 时转红黑树（查询 O(n) → O(log n)），红黑树节点数 &lt; 6 时退化为链表。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">扩容流程：</b>默认容量 16、负载因子 0.75，元素数 &gt; 容量 × 0.75 时扩容为<b>原来的 2 倍</b>（容量恒为 2 的幂，保证位运算定位）。JDK 8 扩容后元素<b>要么留在原位、要么移动到「原位置 + 旧容量」</b>（按新增 bit 是 0 还是 1 拆分），不需要重新计算 hash——这是 JDK 8 对 JDK 7 的重要优化。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>为什么用 2 的幂（<code>hash & (n-1)</code> 等价取模且更快）；扰动函数目的（让高位参与定位，减少冲突）；默认负载因子 0.75 是空间与时间的折中。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>HashMap 线程不安全（并发 put 可能覆盖丢失数据）；自定义对象做 key 必须同时正确实现 <code>hashCode</code> 和 <code>equals</code>（先比 hash 再比 equals）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>HashMap 遍历有序还是随机（无序，插入顺序不保证）；JDK 7 头插法为什么有死循环（扩容时链表反转成环）；Java 17 插入/查找时间复杂度（平均 O(1)，最坏 O(log n)）。</div>
  </div>
</div>

---

# 1.2 HashMap 为什么线程不安全？ConcurrentHashMap 如何保证线程安全？哪些操作是 CAS、哪些上锁？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：转转、腾讯云智、字节、用友</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">HashMap 为什么不安全：</b>① 并发 put 时两个线程算到同一桶，后写覆盖前写（丢数据）；② 扩容期间多线程同时 rehash，JDK 7 头插法可能形成环形链表导致 get 死循环；③ 修改计数（modCount）竞态导致迭代时快速失败。没有任何同步机制。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">ConcurrentHashMap（JDK 8）实现：</b>采用 <b>CAS + synchronized 锁桶头节点</b>，写路径分四步：</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">桶为空 → CAS 直接插入</b>（无锁路径，期望 null 才写入）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">桶非空 → synchronized 锁桶头节点</b>再操作——锁粒度 = 单个桶，多线程可并发写不同桶。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">扩容由多线程协助</b>（transfer 迁移，分担 rehash）。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">size 统计用 CounterCell 分段累加</b>。JDK 7 则是 Segment 分段锁（继承 ReentrantLock）。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>CAS 比较的是<b>桶位是否为 null</b>（期望 null 才插入）；<b>读操作不加锁</b>——Node 的 val 和 next 是 <code>volatile</code>，保证可见性；锁只锁头节点，链表/树内操作在锁内完成。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>CAS 失败怎么办（自旋重试）；为什么读不用锁也不会读到脏数据（volatile + 不变性设计）；和 Hashtable 的区别（全表锁 vs 桶锁，并发度差异）。</div>
  </div>
</div>

---

# 1.3 ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：字节、得物、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">ArrayList（动态数组）</b>
        <div style="margin:4px 0 0;">底层 Object[]，随机访问 <code>O(1)</code>；尾部插入摊还 <code>O(1)</code>（扩容 1.5 倍）；<b>中间插入/删除 O(n)</b>（元素搬移）。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>扩容时复制数组；删除后不缩容。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">LinkedList（双向链表）</b>
        <div style="margin:4px 0 0;">头尾操作 <code>O(1)</code>；<b>中间插入定位 O(n)</b>（要遍历找节点），找到后插入 O(1)。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>每个节点存前后指针，内存占用高；随机访问 O(n)；缓存不友好（节点分散）。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">中间插入优化：</b>① 高频中间插入场景换 <b>LinkedList</b> 或 <b>CopyOnWriteArrayList</b>（写多场景不适用）；② 批量插入用 <code>addAll</code>（一次搬移）；③ 大量中间操作考虑分段结构（如 <code>ArrayDeque</code> 做队列、跳表做有序插入）；④ 先收集到临时列表再整体合并，避免逐条 insert。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么实际项目 ArrayList 用得更多（随机访问 + 缓存友好 + 尾部追加为主）；LinkedList 真的适合队列吗（ArrayDeque 更优，LinkedList 有节点开销）；ArrayList 扩容机制（1.5 倍 + Arrays.copyOf）。</div>
  </div>
</div>

---

# 1.4 哪些集合是线程安全的？怎么把 ArrayList 变线程安全？说说 CopyOnWriteArrayList。

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：招银、字节、熙牛医疗</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">线程安全集合：</b>① <b>遗留类</b>：<code>Hashtable</code>、<code>Vector</code>（全方法 synchronized，性能差）；② <b>并发包</b>：<code>ConcurrentHashMap</code>、<code>CopyOnWriteArrayList</code>、<code>CopyOnWriteArraySet</code>、<code>ConcurrentLinkedQueue</code>、<code>BlockingQueue</code> 系列；③ <b>包装类</b>：<code>Collections.synchronizedList/list/map</code>（包装后方法加锁）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">CopyOnWriteArrayList 原理（写时复制）：</b>写操作（add/set/remove）先<b>复制一份新数组</b>，在副本上修改，然后用 <code>volatile</code> 数组引用替换旧数组；<b>读操作完全不加锁</b>（直接读当前数组引用）。适合<b>读多写少</b>场景（如监听器列表、配置缓存）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>迭代器是<b>快照式</b>（遍历开始时的数组），迭代期间其他线程的修改不可见——弱一致性；写操作 O(n) 复制成本高，写频繁场景性能差；元素不能为 null？可以（允许 null）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>写多读少别用（每次写全量复制）；synchronizedList 读写都加锁（读并发差）；选型：读多写少 → COW，写多 → ConcurrentHashMap/synchronizedList。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>COW 的读操作有没有锁（没有）；和 synchronizedList 的区别（读无锁 vs 读加锁）；为什么迭代器不抛 ConcurrentModificationException（快照迭代）。</div>
  </div>
</div>

## 二、并发编程基础

---

# 2.1 乐观锁与悲观锁的核心区别是什么？分别如何实现（CAS、`synchronized`），各自适用于什么场景？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
<b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">乐观锁</b>
        <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">理念：</b>认为并发冲突发生的概率较低，读数据时不加锁，更新时再检查数据是否被修改过。</div>
        <div style="margin:6px 0 0;"><b style="color:#3F8C12;">实现：</b>CAS：线程带着期望值和新值执行比较并交换，只有当前值仍等于期望值时才更新；数据库中也常用版本号或更新时间字段实现。</div>
        <div style="margin:6px 0 0;"><b style="color:#B26E00;">适用与权衡：</b>适合读多写少、冲突较少的场景，优点是不会因为加锁阻塞线程；但高并发冲突时会频繁重试，CAS 还要注意 ABA 问题。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#B26E00;">悲观锁</b>
        <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">理念：</b>认为冲突概率较高，访问共享资源前先加锁，保证同一时间只有符合条件的线程能够操作。</div>
        <div style="margin:6px 0 0;"><b style="color:#3F8C12;">实现：</b>在 Java 中可以使用 <code>synchronized</code>、<code>ReentrantLock</code>，数据库中则可以使用行锁等。</div>
        <div style="margin:6px 0 0;"><b style="color:#B26E00;">适用与权衡：</b>适合写多、临界区较长或冲突严重的场景，能够降低重试成本，但会带来线程阻塞、上下文切换以及死锁风险。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;padding:8px 12px;border-left:3px solid var(--primary);background:rgba(58,95,191,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">CAS 底层机制：</b>依赖 <code>volatile</code> 保证可见性，通过 <code>Unsafe.compareAndSwapInt</code> 调用 CPU 原子指令（cmpxchg）：比较内存当前值与期望值，相等才写入新值，整个过程原子完成。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">ABA 问题</b><br>值从 A 变 B 又变回 A，CAS 误判未修改；可用 <code>AtomicStampedReference</code> 加版本号解决。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">自旋开销</b><br>冲突激烈时循环重试空转 CPU，高竞争下实际退化为类似锁的串行执行。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">单变量原子性</b><br>只能保证一个共享变量的原子操作，多变量需加锁或封装成对象再整体 CAS。</div>
    </div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #FAAD14;background:rgba(250,173,20,.10);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">常见追问：</b><code>synchronized</code> 锁可以升级：无锁 → 偏向锁 → 轻量级锁 → 重量级锁，竞争加剧时逐级升级，避免一上来就依赖操作系统互斥量；<code>ReentrantLock</code> 则基于 AQS 队列实现，支持可中断、超时、公平锁等更细粒度控制。</div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>CAS 是一种无锁算法实现，不代表没有竞争；<code>synchronized</code> 是基于对象监视器的互斥锁，必要时会让线程阻塞。</div>
</div>

---

# 2.2 synchronized 锁升级：无锁 → 偏向锁 → 轻量级锁 → 重量级锁

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><code>synchronized</code> 在 JDK 1.6 之前是纯重量级锁，直接依赖操作系统互斥量，线程阻塞与唤醒需要用户态/内核态切换，开销大。JDK 1.6 引入锁升级机制：根据竞争激烈程度，锁状态按 无锁 → 偏向锁 → 轻量级锁 → 重量级锁 单向升级。</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
          <b style="color:#3A5FBF;">无锁</b>
          <div style="margin:4px 0 0;">对象未被竞争，Mark Word （对象持有） 保持无锁状态（状态位 01），是所有锁状态的起点。</div>
        </div>
      </div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
          <b style="color:#3A5FBF;">偏向锁</b>
          <div style="margin:4px 0 0;"><b style="color:#3A5FBF;">适用：</b>只有一个线程反复进入同步块。</div>
          <div style="margin:4px 0 0;"><b style="color:#3F8C12;">原理：</b>Mark Word 记录持有线程 ID，该线程再次进入时无需任何原子操作，直接执行。</div>
          <div style="margin:4px 0 0;"><b style="color:#B26E00;">关键：</b>其他线程竞争时撤销偏向锁，撤销需等待安全点（STW）；JDK 15 起默认禁用偏向锁（JEP 374）。</div>
        </div>
      </div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
          <b style="color:#3A5FBF;">轻量级锁</b>
          <div style="margin:4px 0 0;"><b style="color:#3A5FBF;">适用：</b>多个线程交替访问、竞争不激烈。</div>
          <div style="margin:4px 0 0;"><b style="color:#3F8C12;">原理：</b>线程栈帧创建 Lock Record，用 CAS 把 Mark Word 替换为指向锁记录的指针；CAS 失败说明有竞争，自旋等待。</div>
          <div style="margin:4px 0 0;"><b style="color:#B26E00;">关键：</b>自旋避免线程阻塞（JDK 1.6 后为自适应自旋），适合临界区执行时间短的场景。</div>
        </div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
          <b style="color:#B26E00;">重量级锁</b>
          <div style="margin:4px 0 0;"><b style="color:#3A5FBF;">适用：</b>竞争激烈或临界区执行时间长。</div>
          <div style="margin:4px 0 0;"><b style="color:#3F8C12;">原理：</b>锁膨胀为 ObjectMonitor，线程进入阻塞队列等待，依赖操作系统互斥量，涉及用户态/内核态切换。</div>
          <div style="margin:4px 0 0;"><b style="color:#B26E00;">关键：</b>阻塞不消耗 CPU，但唤醒开销大。</div>
        </div>
      </div>
    </div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid var(--primary);background:rgba(58,95,191,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">Mark Word 与状态位：</b>对象头中的 Mark Word 随锁状态变化——偏向锁存线程 ID、轻量级锁存锁记录指针、重量级锁存 Monitor 指针；状态标志位：01 无锁/偏向、00 轻量级、10 重量级、11 GC 标记。</div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #FAAD14;background:rgba(250,173,20,.10);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">常见追问：</b>① 锁只能升级不能降级（降级仅可能发生在安全点停顿的极端场景）；② 偏向锁撤销需要 STW 是 JDK 15 默认禁用它的原因（JEP 374）；③ 自旋是自适应的——根据上次自旋结果动态调整次数，避免白白空转 CPU。</div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>锁升级的本质是“竞争越激烈，锁越重”：一个线程反复进 → 偏向；多线程交替进 → 轻量级（CAS + 自旋）；竞争激烈或临界区长 → 重量级（Monitor 阻塞）。</div>
</div>

---

# 2.3 synchronized 和 ReentrantLock 有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：浩鲸、用友、字节、转转</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">synchronized</b>
        <div style="margin:4px 0 0;">JVM 关键字，基于 Monitor，锁升级（偏向→轻量→重量）；使用简单（方法/代码块）；自动释放锁（异常也释放）。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">局限：</b>不能响应中断、不能超时、非公平、不可见锁状态、获取后不能手动控制条件。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">ReentrantLock</b>
        <div style="margin:4px 0 0;">JDK 类，基于 <b>AQS</b>（CLH 队列 + CAS 状态）。支持：<b>可中断</b>（<code>lockInterruptibly</code>）、<b>超时</b>（<code>tryLock(timeout)</code>）、<b>公平锁</b>（构造参数）、<b>多个 Condition</b>（精确唤醒）、可查询锁状态。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>必须手动 <code>unlock</code>（finally 中释放）；API 复杂。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">怎么选：</b>简单互斥用 <code>synchronized</code>（简洁 + 自动释放）；需要超时/中断/公平/多条件（如生产者消费者精确唤醒）用 <code>ReentrantLock</code>；两者都是可重入锁。底层都支持锁升级/自旋优化。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>可重入怎么实现（AQS 的 state 计数）；公平锁为什么慢（队列唤醒 + 上下文切换）；Condition 和 wait/notify 的区别（多条件精确唤醒 vs 全量唤醒）。</div>
  </div>
</div>

---

# 2.4 volatile 的作用？为什么只能保证可见性和有序性、保证不了原子性？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯云智、振心</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b><code>volatile</code> 保证<b>可见性</b>（写 volatile 变量立即刷新主存，读强制从主存拿，其他线程立即可见）和<b>有序性</b>（禁止指令重排，内存屏障）。JMM 模型：线程有工作内存（寄存器/缓存），主存共享变量——volatile 强制读写穿透主存。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">为什么保证不了原子性：</b>volatile 不管<b>复合操作</b>。以 <code>i++</code> 为例：读 i → 加 1 → 写回 i，三步都可能被其他线程穿插——volatile 只保证每步的可见性，不保证三步作为一个整体不被拆开。原子性要靠 <code>synchronized</code> 或 <code>AtomicInteger</code>（CAS）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">典型应用：</b>① 状态标志位（<code>volatile boolean running</code>，一个线程写、其他线程读）；② 双重检查锁单例（<code>DCL</code> 中的 volatile 防止 <code>new</code> 三步重排暴露半初始化对象）；③ 无锁队列的写指针。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>懒汉单例里 volatile 有什么用（防止分配内存→初始化→赋值的重排）；JMM 主存和工作内存怎么同步（read/load/use/assign/store/write 八种操作 + volatile/锁触发屏障）；volatile 和 synchronized 的区别（可见性+有序 vs 三性全保）。</div>
  </div>
</div>

---

# 2.5 ThreadLocal 的用法、原理和注意事项？为什么 key 用弱引用？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：字节、腾讯云智、阿里飞猪</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>ThreadLocal 为每个线程保存一份<b>独立副本</b>，线程间互不干扰。原理：每个 Thread 内部有 <code>ThreadLocalMap</code>，key 是 ThreadLocal 对象（弱引用），value 是线程持有的副本值。读写走 <code>Thread.currentThread().threadLocals</code>，天然线程隔离。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">为什么 key 弱引用：</b>key（ThreadLocal 对象）通常只被外部引用一次，如果 key 是强引用，外部 ThreadLocal 置 null 后 key 永远无法回收——<b>弱引用让 key 可被 GC</b>。value 是强引用——这就是泄漏来源：key 被回收后 value 还挂在 ThreadLocalMap 里，线程（尤其线程池线程）存活时 value 无法访问也无法回收。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">内存泄漏与避免：</b>既然 key 弱引用了，为什么还泄漏？——<b>value 是强引用</b>，线程池线程长期存活时，key 为 null 的 Entry 不会被自动清（只有 get/set/remove 时顺带清理部分）。规范做法：<b>用完必须 <code>remove()</code></b>（尤其线程池场景，try-finally 包裹）；阿里规范明确要求。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">注意事项：</b>① 线程池中不 remove 必泄漏；② 父子线程不共享（InheritableThreadLocal 可传值但线程池不适用，用 TransmittableThreadLocal）；③ 值别放重量级对象（每线程一份）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>ThreadLocal 和 synchronized 的区别（空间换时间 vs 互斥）；为什么线程池里更危险（线程复用存活久）；怎么避免（remove + 合理生命周期）。</div>
  </div>
</div>

---

# 2.6 Java 内存模型（JMM）：主存和工作内存是什么？两者怎么同步？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：字节、腾讯云智</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>JMM（Java 内存模型）描述线程与内存的抽象关系：<b>主内存</b>（所有线程共享的变量存储区）和<b>工作内存</b>（线程私有的变量副本，含寄存器/缓存）。线程不能直接操作主内存，必须先读入工作内存再操作，操作完再写回主内存——这就是并发可见性问题的根源。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实现流程（八种原子操作）：</b>主内存 → 工作内存：<code>read</code>（主存读）→ <code>load</code>（载入工作内存）→ <code>use</code>（使用）；工作内存 → 主内存：<code>assign</code>（赋值）→ <code>store</code>（存）→ <code>write</code>（写回主存）；另有 <code>lock</code>（加锁）和 <code>unlock</code>（解锁）作用于主内存变量。同步靠这些操作按规则配对执行。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">同步触发条件（何时强制穿透）：</b>① <b>volatile</b>：写 = 强制 store/write 刷主存，读 = 强制 read/load 从主存取；② <b>锁（synchronized/Lock）</b>：加锁时清空工作内存，解锁时把修改刷回主存；③ <b>final</b>：构造完成后其他线程可见。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>没有同步机制时，线程 A 的修改可能长时间不写回主存（可见性），或指令重排改变执行顺序（有序性）——这就是 <code>volatile</code>/锁要解决的两件事；但 JMM 是<b>抽象模型</b>，实际由内存屏障 + 缓存一致性协议（MESI 等）在硬件层实现。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>JMM 和 JVM 内存区域的关系（抽象并发模型 vs 运行时区域，两者独立）；happens-before 规则是什么（先行发生原则，volatile/锁/线程启动等）；为什么加锁要清空工作内存（保证读到最新值）。</div>
  </div>
</div>

---

# 2.7 什么是 AQS？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>AQS 是 JUC 的同步器框架，核心是 <b>state + CLH 等待队列 + CAS + park/unpark 唤醒</b>——把并发控制从「每个锁自己造轮子」变成「只实现状态规则，其余排队唤醒交给框架」。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">核心组成：</b><code>state</code> 一个 int 状态位（锁是否被占用、剩余许可数）；等待队列是 CLH 变种双向链表，抢不到资源的线程排队；CAS 原子修改 state；<code>park/unpark</code> 挂起与唤醒线程。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">独占锁工作流程：</b>线程先 CAS 抢 state → 抢到进临界区 → 抢不到入队并 park 挂起 → 持有线程释放后 unpark 队头线程 → 被唤醒线程再尝试 CAS 抢锁。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实现方式：</b>只需重写 <code>tryAcquire/tryRelease</code>（独占）或 <code>tryAcquireShared/tryReleaseShared</code>（共享）——AQS 负责排队、阻塞、唤醒，你只定义规则。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">基于 AQS 的组件：</b>① <b>ReentrantLock</b>：state = 重入次数，同一时刻只允许一个线程持有；② <b>Semaphore</b>：state = 剩余许可数，获取 -1、释放 +1，最多 N 个线程同时通过；③ <b>CountDownLatch</b>：state = 剩余计数，到 0 一次性放开，不可重置（重置用 CyclicBarrier/Phaser）；④ <b>ReentrantReadWriteLock</b>：state 高低位分别编码读锁和写锁计数。</div>
  </div>
</div>

---

# 2.8 死锁如何排查？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">先说定义：</b>死锁是多个线程互相持有对方需要的资源，并且永久等待。经典死锁同时满足<b>互斥、占有且等待、不可剥夺、循环等待</b>四个条件；排查时要先证明存在环路，再定位业务代码中的加锁顺序。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">线上排查流程：</b>① 记录进程号、线程数、CPU 和接口延迟，确认是死锁而不是普通锁竞争或下游阻塞；② 使用 <code>jstack &lt;pid&gt;</code>、<code>jcmd &lt;pid&gt; Thread.print -l</code> 连续采集两到三次线程转储，观察线程状态和锁等待是否持续；③ 搜索 <code>Found one Java-level deadlock</code>、<code>BLOCKED</code>、<code>waiting to lock</code>，沿着“线程 A 持有锁 X 等锁 Y、线程 B 持有锁 Y 等锁 X”的关系还原环；④ 回到代码核对锁对象是否一致、是否存在嵌套加锁和异常分支未释放资源。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">工具边界：</b><code>jstack</code> 对 <code>synchronized</code> 和 JVM monitor 最直观；<code>ReentrantLock</code> 要结合线程栈、<code>getOwner</code>、锁等待日志或 JFR 事件判断；数据库锁、Redis 分布式锁还需要查看数据库锁表、Redis key 的持有者和租约信息，不能只看 JVM 线程转储。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">修复优先级：</b>① 对多个资源规定全局一致的加锁顺序，例如按账户 ID 从小到大加锁；② 尽量缩短临界区，锁内不做网络调用、磁盘 I/O、复杂计算和用户回调；③ 对可中断场景使用 <code>tryLock(timeout)</code>，超时后释放已持有的锁并返回可重试结果；④ 使用 <code>try/finally</code> 保证释放，避免锁对象被重新赋值；⑤ 对跨进程锁设置租约、续期和持有者标识，防止客户端异常后永久占用。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">注意事项：</b>不要只通过重启进程掩盖问题；重启可以止血，但还要保留线程转储、请求链路和锁指标。预防上可监控锁等待时长、BLOCKED 线程数、接口超时率，并在压测和故障演练中覆盖嵌套锁、超时和异常路径。</div>
    <div style="margin:8px 0 0;"><b>面试追问：</b>“为什么线程 dump 没发现死锁？”可能是锁在数据库或其他 JVM、采集时机已错过，或者只是长时间阻塞；“如何避免死锁又保证并发？”通常采用统一顺序、缩小临界区和分段锁，并用超时机制限制最坏等待时间。</div>
  </div>
</div>

---

# 2.9 CompletableFuture 常见坑？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心认识：</b><code>CompletableFuture</code> 是一个可组合的异步结果容器，不等于“自动异步”。不带 executor 的 <code>async</code> 方法通常使用公共 <code>ForkJoinPool.commonPool()</code>，而不带 <code>async</code> 的回调可能直接在完成前一个阶段的线程上执行。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">组合语义：</b>① <code>thenApply</code> 是结果转换，返回普通值；② <code>thenCompose</code> 用于把两个异步阶段串联，避免产生嵌套的 <code>CompletableFuture&lt;CompletableFuture&lt;T&gt;&gt;</code>；③ <code>thenCombine</code> 等待两个独立任务后合并结果；④ <code>allOf</code> 只表示全部完成，结果需要从原 future 中收集，不能直接得到列表。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">最常见的坑：</b>① 在回调中调用 <code>join/get</code>，阻塞公共线程池，形成线程饥饿；② 把 CPU 密集型任务和网络、数据库等阻塞任务放在同一个线程池；③ 忽略返回的新 future，导致异常或转换结果丢失；④ 误以为 <code>whenComplete</code> 能吞掉异常，实际上它主要用于观察，异常仍会继续传播；⑤ 异常只在最终 <code>join</code> 时暴露，定位距离真实故障点很远。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">异常与超时：</b>用 <code>exceptionally</code> 提供降级值，用 <code>handle</code> 同时处理正常值和异常，用 <code>whenComplete</code> 记录日志和指标；对外部调用使用 <code>orTimeout</code> 控制最长等待，允许降级时使用 <code>completeOnTimeout</code> 返回兜底值。超时不一定能中断底层 HTTP 或数据库调用，底层客户端仍需配置连接、读写和总超时。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">线程池与上下文：</b>为阻塞 I/O、CPU 计算和低延迟任务配置独立 executor，设置有界队列、拒绝策略和线程命名；在异步边界显式传递 traceId、用户身份和 MDC，上下文不能依赖 ThreadLocal 自动继承。提交任务前还要限制并发度，避免 <code>allOf</code> 一次性扇出过多请求。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">重试与取消：</b>重试必须限定异常类型、次数和退避时间，并确保操作幂等；<code>cancel</code> 主要完成 future 的状态变更，不保证已经执行的底层任务被真正终止，因此需要结合任务中断响应和客户端取消接口。生产环境应记录每个阶段耗时、排队长度、成功率、异常类型和超时数量。</div>
    <div style="margin:8px 0 0;"><b>面试追问：</b>“为什么不用一条链全部异步？”异步只能隐藏等待，不能消除下游容量和线程池约束；“如何保证结果顺序？”为每个任务保留输入序号，完成后按序收集；“如何避免级联超时？”给整条链路设置总预算，再为每个子调用分配更短的剩余超时，并在超时后及时停止继续扇出。</div>
  </div>
</div>

## 三、线程池

---

# 3.1 线程池提交任务后的核心执行流程是什么？请说明完整处理顺序。

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">以 <code>ThreadPoolExecutor.execute()</code> 为例，提交任务后的顺序是：</div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">如果当前工作线程数小于 <code>corePoolSize</code>，优先创建核心线程执行任务。</div>
      </div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">如果核心线程已达到上限，并且线程池仍在运行，就尝试把任务放入阻塞队列。</div>
      </div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">入队后会再次检查线程池状态：如果线程池已经停止，则移除任务并执行拒绝策略；如果线程数变成零，则补充一个工作线程。</div>
      </div>
      <div style="position:relative;margin:0 0 10px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">如果队列已满，且当前线程数小于 <code>maximumPoolSize</code>，创建非核心线程执行任务。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">5</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">如果线程数也达到最大值，则执行 <code>RejectedExecutionHandler</code>，例如直接拒绝、调用方执行或丢弃任务。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid var(--primary);background:rgba(58,95,191,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">七个核心参数：</b><code>corePoolSize</code> 核心线程数、<code>maximumPoolSize</code> 最大线程数、<code>keepAliveTime</code> 空闲存活时间、<code>unit</code> 时间单位、<code>workQueue</code> 阻塞队列、<code>threadFactory</code> 线程工厂、<code>handler</code> 拒绝策略；<code>submit()</code> 底层仍是 <code>execute()</code>，只是包装了 <code>Future</code> 返回结果。</div>
    <div style="margin:8px 0 0;">队列满且线程数达上限时，由 <code>RejectedExecutionHandler</code> 决定拒绝策略：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">AbortPolicy（默认）</b><br>直接抛 <code>RejectedExecutionException</code>。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">CallerRunsPolicy</b><br>由提交任务的线程自己执行，天然限流、不会丢任务。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">DiscardPolicy</b><br>静默丢弃新任务，不报错。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">DiscardOldestPolicy</b><br>丢弃队列中最老的任务，再重试提交新任务。</div>
    </div>
    <div style="margin:10px 0 0;">工作线程启动后，会循环从队列获取任务，依次执行 <code>beforeExecute</code>、任务本身和 <code>afterExecute</code>；线程退出时更新完成任务数，并由线程池处理工作线程的回收或补充。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>线程池的优先级是“核心线程 → 任务队列 → 非核心线程 → 拒绝策略”，不是提交一个任务就立即创建新线程。</div>
</div>

---

# 3.2 线程池如何通过 `keepAliveTime`（非核心线程空闲多久后被销毁） 实现空闲线程的超时自动销毁？其核心逻辑是什么？


<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">核心原理：</b><code>keepAliveTime</code> 的实现核心不是“线程池启动一个定时器，定期扫描线程并销毁”，而是<b>空闲线程自己在任务队列上做带超时时间的阻塞等待；等待超过 <code>keepAliveTime</code> 仍然拿不到任务，就让这个 Worker 自己退出</b>。</div>
    <div style="margin:8px 0 0;">线程池工作线程在 <code>getTask()</code> 中获取任务。对于非核心线程，或者开启了 <code>allowCoreThreadTimeOut</code> 的核心线程，会使用带超时时间的 <code>poll(keepAliveTime)</code>；对于默认核心线程，则通常使用不会超时的 <code>take()</code>。</div>
    <div style="margin:8px 0 0;">如果在 <code>keepAliveTime</code> 内没有取到任务，<code>poll</code> 返回空，线程会被判定为超时。线程池随后减少工作线程数量，让 <code>runWorker</code> 结束，空闲线程就自动销毁。只要线程数仍低于核心线程数，默认情况下就不会回收核心线程；开启 <code>allowCoreThreadTimeOut(true)</code> 后，核心线程也会按这个规则回收。</div>
    <div style="margin:8px 0 0;">因此，<code>keepAliveTime</code> 控制的是“空闲线程等待新任务的最长时间”，不是任务执行超时时间。它主要用于在流量下降时释放非核心线程占用的资源，在流量恢复时再按需创建线程。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #FAAD14;background:rgba(250,173,20,.10);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">常见追问：</b>① 回收下限是 <code>corePoolSize</code>（开启 <code>allowCoreThreadTimeOut(true)</code> 后下限为 0），核心线程默认用 <code>take()</code> 阻塞等待、不参与超时回收，避免频繁创建销毁线程的开销；② 开启核心线程超时回收时 <code>keepAliveTime</code> 必须大于 0；③ 正在执行任务的线程不会因为超时被强制销毁，超时只作用于取任务的等待过程。</div>
</div>

---

# 3.3 线程池的拒绝策略是什么

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">触发时机：<code>execute()</code> 提交任务时，若阻塞队列已满（入队失败）且当前工作线程数已达到 <code>maximumPoolSize</code>，线程池会调用 <code>RejectedExecutionHandler.rejectedExecution()</code> 处理该任务。JDK 内置四种策略：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:var(--primary);">AbortPolicy（默认）</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">行为：</b>直接抛 <code>RejectedExecutionException</code>。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>默认策略，任务会直接丢失，调用方需捕获异常或靠监控告警发现。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:var(--primary);">CallerRunsPolicy</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">行为：</b>由提交任务的线程自己执行该任务，不丢任务。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>天然限流——提交线程被任务占用，后续提交自然放慢；但慢任务会阻塞调用方。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:var(--primary);">DiscardPolicy</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">行为：</b>静默丢弃新任务，不抛异常。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>丢弃无感知，适合允许丢消息的场景，需配合监控指标。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:var(--primary);">DiscardOldestPolicy</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">行为：</b>丢弃队列中最老的任务，再重试提交新任务。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>配优先级队列时会丢弃优先级最高的任务，与直觉相反；被丢的可能是关键业务。</div>
      </div>
    </div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid var(--primary);background:rgba(58,95,191,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">生产实践：</b>核心链路常用 <code>AbortPolicy</code> + 监控告警（fail-fast，暴露问题）；能容忍延迟或丢消息的场景用 <code>CallerRunsPolicy</code> 或自定义策略；<code>DiscardPolicy</code>/<code>DiscardOldestPolicy</code> 用得少，因为静默丢任务难以发现。</div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #FAAD14;background:rgba(250,173,20,.10);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">常见追问：</b>① 如何自定义策略？实现 <code>RejectedExecutionHandler</code> 接口，常见做法：降级返回兜底值、带上限重试入队、写入本地文件或消息队列补偿、延迟后重试；② <code>DiscardOldestPolicy</code> 的坑：配合 <code>PriorityBlockingQueue</code> 时丢的是最高优先级任务；③ <code>CallerRunsPolicy</code> 会阻塞提交线程，调用方不可接受阻塞的场景要慎用。</div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>拒绝策略的本质是“任务饱和时丢谁、怎么丢”：AbortPolicy 抛异常 fail-fast、CallerRunsPolicy 不丢但反压提交方、两个 Discard 分别丢新/旧任务；生产常用 AbortPolicy + 告警，或自定义策略做降级补偿。</div>
</div>

---

# 3.4 为什么不建议直接用 Executors 创建线程池？线上任务堆积如何排查？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：浩鲸、熙牛医疗、字节、叶子科技</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">为什么不建议 Executors：</b>① <code>newFixedThreadPool</code>/<code>newSingleThreadExecutor</code> 用<b>无界 LinkedBlockingQueue</b>——任务无限堆积，内存 OOM 且拒绝策略永远不触发；② <code>newCachedThreadPool</code> 最大线程数 <b>Integer.MAX_VALUE</b>——线程无限创建，CPU/内存耗尽；③ <code>newScheduledThreadPool</code> 同样无界。规范做法：<code>new ThreadPoolExecutor(...)</code> 手动指定<b>有界队列</b>（ArrayBlockingQueue）+ 明确拒绝策略 + 线程工厂命名。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">线上任务堆积排查流程：</b>① 看指标：队列长度（<code>getQueue().size()</code>）、活跃线程数、拒绝次数、任务耗时 P99；② 判断方向：堆积 = 消费慢（任务执行慢/下游慢）还是生产快（流量突增）；③ 下钻：任务在等什么——数据库慢、RPC 超时、锁等待、GC；④ 处理：先扩容消费者/临时加线程（限流保护下游），再修根因（优化任务、限流入口）；⑤ 监控告警：队列水位、任务拒绝率、线程池活跃度。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">核心参数怎么定：</b>CPU 密集 = <code>CPU 核数 + 1</code>；I/O 密集 = <code>核数 × (1 + 等待时间/计算时间)</code>（经验值 2 倍左右）；队列大小按峰值积压容忍度；拒绝策略按业务（核心链路 AbortPolicy + 告警，可降级场景 CallerRunsPolicy）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>线程池监控哪些指标（活跃线程/队列/拒绝/完成任务数）；堆积时先扩线程还是先查根因（先限流保命再查根因）；CallerRunsPolicy 在堆积时有什么效果（反压调用方）。</div>
  </div>
</div>

---

# 3.5 为什么常用有界队列？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#B26E00;">无界队列的坑：</b>看似平滑，实则在高峰会<b>无限堆积任务</b>，导致内存和时延失控；任务等待时间越来越长，最终可能触发 OOM 或请求级联超时。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">有界队列的价值：</b>能把压力<b>显式暴露</b>出来，配合拒绝策略和降级更符合生产治理思路。当队列接近上限时，可以快速失败、调用方执行、丢弃低优先级任务或触发限流，而不是让所有请求无限等待。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">容量怎么定：</b>根据任务平均耗时、目标吞吐、最大可接受排队时延和线程数估算，再通过压测校准；队列越长不代表吞吐越高，还要监控队列长度、最老任务年龄、拒绝次数、线程池活跃数和任务耗时。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">取舍：</b>队列太小会造成频繁拒绝，队列太大又会隐藏下游瓶颈。不同优先级或不同下游依赖应使用隔离队列，不能让低价值任务占满核心任务的容量。</div>
  </div>
</div>

## 四、JVM

---

# 4.1 JVM 运行时数据区域如何划分？堆、虚拟机栈、方法区等区域分别有什么作用？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <div style="margin:6px 0 0;">JVM 运行时数据区域可以分为线程私有区域和线程共享区域：</div>
    <div style="display:grid;grid-template-columns:72px 1fr;column-gap:12px;row-gap:8px;margin:8px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">程序计数器</div>
      <div>线程私有，记录当前线程下一条要执行的字节码位置，是唯一不会发生 OOM 的区域。</div>
      <div style="color:#3A5FBF;font-weight:600;">虚拟机栈</div>
      <div>线程私有，每次方法调用创建一个栈帧，栈帧包含局部变量表、操作数栈、动态链接和方法返回地址；栈深度超限抛 <code>StackOverflowError</code>。</div>
      <div style="color:#3A5FBF;font-weight:600;">本地方法栈</div>
      <div>线程私有，为 Native 方法调用服务。</div>
      <div style="color:#3A5FBF;font-weight:600;">堆</div>
      <div>线程共享，存放对象实例和数组，是垃圾回收的主要区域，对象过多时抛 <code>OutOfMemoryError: Java heap space</code>。</div>
      <div style="color:#3A5FBF;font-weight:600;">方法区</div>
      <div>线程共享，存放类元信息、运行时常量池、静态变量等。Java 8 以后 HotSpot 主要使用元空间实现方法区，元空间使用本地内存、默认无上限，类元信息过多会抛 <code>OOM: Metaspace</code>。</div>
    </div>
    <div style="margin:10px 0 0;">直接内存不属于 JVM 规范规定的运行时数据区域，但 <code>NIO</code> 等场景会使用它，排查内存问题时也需要关注。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">栈溢出</b><br>递归过深或栈帧过多 → <code>StackOverflowError</code>。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">堆 OOM</b><br>对象实例过多 → <code>OutOfMemoryError: Java heap space</code>。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">元空间 / 直接内存 OOM</b><br>类元信息过多 → <code>OOM: Metaspace</code>；NIO 缓冲区过多 → <code>OOM: Direct buffer memory</code>。</div>
    </div>
  </div>
</div>

---

# 4.2 Java 内存泄漏的常见原因有哪些？如何排查？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">内存泄漏是指对象已经不再需要，但仍然被 GC Roots 引用，导致垃圾回收器无法回收。常见原因包括：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 200px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;">静态集合长期持有对象</div>
      <div style="flex:1 1 200px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;">缓存没有淘汰策略</div>
      <div style="flex:1 1 200px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;">监听器或回调没有解绑</div>
      <div style="flex:1 1 200px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;">线程池中的 <code>ThreadLocal</code> 没有及时 <code>remove</code></div>
      <div style="flex:1 1 200px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;">连接和流等资源没有关闭</div>
      <div style="flex:1 1 200px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;">类加载器泄漏</div>
    </div>
    <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #FAAD14;background:rgba(250,173,20,.10);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">常见追问：</b>为什么 <code>ThreadLocal</code> 会泄漏？<code>ThreadLocalMap</code> 的 Entry 对 key（ThreadLocal 对象）是弱引用，但对 value 是强引用；线程池线程长期存活时，key 被回收而 value 无人取出，value 就再也无法被访问 → 泄漏。规范做法是每次用完后 <code>remove()</code>，<code>set/get</code> 时也会顺带清理部分过期 Entry。</div>
    <div style="margin:10px 0 0;">排查流程：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">① 判断是否真泄漏</b><br>先确认是泄漏还是业务本身确实需要更多内存，通过监控观察堆使用量和 GC 后老年代是否持续上涨。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">② dump 定位持有者</b><br>接近 OOM 时导出多份 heap dump，用 MAT、VisualVM 或 JProfiler 查看支配树、对象数量和 GC Roots 引用链；结合线程 dump、GC 日志和 JFR 确认创建与释放路径。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">③ 修复后验证</b><br>通过重复场景和多轮 GC 验证对象数量能够回落。</div>
    </div>
  </div>
</div>
---

# 4.3 常见的垃圾回收算法有哪些？请说明标记-清除、复制、标记-整理和分代收集算法的特点。

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;"><b style="color:var(--primary);">标记-清除</b><br>先标记存活对象，再清除不可达对象。实现简单，但需要两次遍历（标记 + 清除），且会产生大量内存碎片。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;"><b style="color:var(--primary);">复制</b><br>把存活对象复制到另一块连续区域，再清空原区域。不会产生碎片，但需要额外空间，适合存活对象较少的区域；HotSpot 新生代默认 Eden:Survivor = 8:1:1，空间浪费率约 10%。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;"><b style="color:var(--primary);">标记-整理</b><br>先标记存活对象，再将存活对象向一端移动，最后清理边界外空间。能够减少碎片，但移动对象成本更高。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;"><b style="color:var(--primary);">分代收集</b><br>根据对象生命周期把堆划分为新生代和老年代，新生代使用更适合大量短命对象的复制思想，老年代使用标记-清除或标记-整理等算法。它利用了大多数对象朝生夕死、少数对象长期存活的规律。</div>
    </div>
    <div style="margin:10px 0 0;">现代垃圾收集器通常是这些算法的组合，例如年轻代复制、老年代标记整理；具体策略还要结合吞吐量、延迟和堆大小选择收集器。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #FAAD14;background:rgba(250,173,20,.10);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">常见追问：</b>为什么老年代不用复制算法？复制依赖“存活率低”的前提，老年代存活率高、复制成本大且需要额外空间；新生代 8:1:1 分区中 Survivor 空间不足时，对象会通过分配担保机制直接进入老年代。</div>
</div>

---

# 4.4 对象什么时候被回收？怎么判断对象已经死亡？引用类型有哪些？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：浩鲸、字节、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理（存活判定）：</b>JVM 用<b>可达性分析</b>（GC Roots 算法）判定对象是否存活：从 GC Roots（栈帧局部变量、静态变量、常量池引用、JNI 引用）出发遍历，<b>不可达</b>的对象判定为可回收。引用计数法（循环引用问题）已被放弃。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">回收流程（两次标记）：</b>不可达对象不会立刻回收——先做第一次标记并判断是否需要 <code>finalize()</code>；需要则进入 F-Queue 等待执行，执行后若仍不可达（或没自救）才真正回收。注意：<b>finalize 不推荐使用</b>（不确定执行时机，现代 JVM 建议直接不用）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">四种引用类型：</b>① <b>强引用</b>：普通 new，只要存在就<b>永不回收</b>（OOM 也不回收）；② <b>软引用（SoftReference）</b>：内存充足不回收，<b>即将 OOM 时回收</b>——适合缓存（图片/大对象）；③ <b>弱引用（WeakReference）</b>：<b>下一次 GC 就回收</b>——ThreadLocal key、WeakHashMap；④ <b>虚引用（PhantomReference）</b>：随时可回收，配合引用队列做<b>对象回收通知</b>（如堆外内存清理）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>软引用和弱引用的区别（OOM 前回收 vs 下次 GC 回收）；引用队列是什么（对象被回收后入队通知）；为什么不用引用计数（循环引用无法回收）。</div>
  </div>
</div>

---

# 4.5 常用垃圾回收器有哪些？CMS 和 G1 怎么选？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：古茗、腾讯CSIG、有赞、滴滴、BIGO</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">常用收集器：</b>① <b>Serial/ParNew</b>：单线程/多线程新生代复制，STW；② <b>Parallel Scavenge + Parallel Old</b>：吞吐优先（默认 JDK 8），多线程并行回收；③ <b>CMS</b>：老年代并发标记清除，追求低停顿（标记-清除，有碎片）；④ <b>G1</b>：JDK 9+ 默认，<b>分区域（Region）</b>化，可预测停顿（-XX:MaxGCPauseMillis），整体标记-整理 + 局部复制，兼顾吞吐与延迟；⑤ <b>ZGC/Shenandoah</b>：超低停顿（毫秒级），大堆场景。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">CMS vs G1：</b>① CMS 基于标记-清除（碎片化，要配合 Full GC 整理），G1 基于 Region + 标记-整理（无碎片问题）；② CMS 并发标记用<b>增量更新</b>，G1 用<b>原始快照（SATB）</b>——都为了处理并发标记期间对象引用变化；③ G1 停顿可预测（按停顿目标选回收 Region），CMS 停顿不可控；④ 大堆（>4-8G）G1 优势明显；⑤ JDK 9 起 CMS 废弃，JDK 14 移除——新项目直接 G1。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">怎么选：</b>追求吞吐（离线计算）→ Parallel；追求低延迟（在线服务）→ G1（默认）；超大堆 + 毫秒级停顿 → ZGC；选型后要压测验证停顿时间与吞吐。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>增量更新和原始快照的区别（记录新增引用 vs 记录删除引用）；G1 的 Remembered Set 是什么（记录 Region 间引用，避免全堆扫描）；CMS 为什么有碎片问题（标记-清除不清整）。</div>
  </div>
</div>

---

# 4.6 线上遇到过 OOM 吗？怎么排查？top/jps/jstack/jmap 各能看什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯云智、字节、致远互联</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>OOM 排查核心：<b>先看错误类型，再抓现场，最后用工具分析</b>。常见类型：<code>Java heap space</code>（堆满）、<code>Metaspace</code>（类元信息）、<code>GC overhead limit exceeded</code>（GC 白忙）、<code>Unable to create native thread</code>（线程数超限）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">排查流程（命令链）：</b></div>
    <div style="position:relative;padding-left:36px;margin:10px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><code>top</code>：看进程 CPU/内存占用，确认哪个进程异常。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><code>jps -l</code>：列出 JVM 进程和主类，找到目标 PID。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><code>jmap -dump</code>：<b>OOM 前抓堆 dump</b>（或启动参数 <code>-XX:+HeapDumpOnOutOfMemoryError</code> 自动抓）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><code>jstack</code>：看线程栈（线程卡死/死锁/大量等待）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">5</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><code>jstat -gcutil</code>：看 GC 频率（是否频繁 Full GC）。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">6</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b>MAT 分析 dump</b>：Dominator Tree 找大对象、Leak Suspects 找泄漏嫌疑、GC Roots 引用链定位持有者。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">7</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">结合代码修复</b>：静态集合、缓存无淘汰、连接未关、ThreadLocal 未 remove。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>无头服务器怎么定位（同命令链，远程 ssh + 工具）；dump 文件可能很大（先 <code>jmap -histo</code> 看对象统计再决定是否全量 dump）；OOM 后进程可能还在（GC overhead）也可能已退出——生产要配自动重启 + dump 保留；只背命令名没用，要能讲清每步解决什么问题。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>heap dump 和 thread dump 的区别（堆对象 vs 线程状态）；jmap -histo 看什么（对象类型统计/占用）；OOM 后第一件事做什么（保留现场：dump + 日志 + 指标快照）。</div>
  </div>
</div>

---

# 4.7 标记-清除算法导致内存碎片化，要分配一块很大的内存怎么办？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>标记-清除只清不整，留下碎片——连续大对象可能找不到足够大的连续空间。JVM 的应对策略：</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">应对机制：</b>① <b>标记-整理（Compact）</b>：CMS 触发 Full GC 时做整理；G1 在回收 Region 时是复制（等效整理），天然无碎片；② <b>大对象直接进老年代</b>：超过阈值（<code>-XX:PretenureSizeThreshold</code>）的大对象直接分配到老年代，避免在新生代反复复制——但老年代也可能碎片化；③ <b>TLAB 与大对象分配</b>：TLAB 放不下的大对象走堆外分配路径；④ <b>内存预留</b>：CMS 预留空间（<code>-XX:CMSInitiatingOccupancyFraction</code>）防碎片导致 promotion failure；⑤ 兜底：老年代空间不足且无法整理时抛 <code>OutOfMemoryError</code> 或触发 Full GC。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>碎片化严重时表现：老年代占用不高但频繁 Full GC（大对象找不到连续空间）——G1/整理型收集器能根治；大对象分配失败先看是不是碎片（<code>-XX:+PrintGCDetails</code> 看 promotion failed / concurrent mode failure）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>大对象为什么直接进老年代（避免新生代复制开销）；promotion failure 是什么（老年代碎片导致晋升失败）；G1 为什么没碎片（Region 复制 + 逻辑连续）。</div>
  </div>
</div>

---

# 4.8 G1 的核心思想是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">① 堆结构变化：</b>把整个堆切成等大小 <b>Region</b>（1MB~32MB），每个 Region 可在不同阶段扮演 Eden/Survivor/Old——不再是年轻代老年代的固定物理分区。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">② 回收核心（Garbage First）：</b>每次 GC 不一定要全堆扫，而是优先选<b>"垃圾占比高、回收收益大"</b>的 Region——用更少的停顿拿到更多可用内存。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">③ 关键流程：</b><code>Young GC</code>（回收 Eden/Survivor，存活对象晋升/复制）→ <code>Concurrent Mark</code>（并发标记全堆存活对象，统计 Region 存活率）→ <code>Mixed GC</code>（年轻代回收时顺带回收高收益 Old Region）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">④ 可预测停顿：</b>给 <code>-XX:MaxGCPauseMillis</code> 目标，G1 根据历史统计（Region 回收/复制成本）动态控制回收集合大小，让停顿贴近目标。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">一句话：</b>G1 的本质是把 GC 从"整代粗粒度回收"变成"<b>Region 粒度、收益驱动、带停顿预算的回收调度器</b>"。</div>
  </div>
</div>

---

# 4.9 JDK 8 升级到 JDK 21 有什么变化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（三点）：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">语言表达力</b><br>语法糖爆炸（record/switch 表达式/文本块），代码量砍 30-50%，可替换 Lombok。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">虚拟线程</b><br>高并发 I/O 降维打击：轻松跑百万级并发，不用堆大线程池。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">GC 与 JVM 性能</b><br>G1 默认 + ZGC/Shenandoah 亚毫秒暂停、JIT 优化 + CDS 默认开启：启动时间砍半、内存占用降 30-50%。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">GC 变化细化：</b>① 默认收集器从 Parallel 变 <b>G1</b>（吞吐 → 吞吐+停顿平衡）；② 低延迟可选 ZGC/Shenandoah（JDK 21 成熟度高）；③ G1 在 9~21 持续优化（并行 Full GC、内存归还机制）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>虚拟线程和线程池的区别（用户态调度 vs 平台线程池）；CDS 是什么（类数据共享，启动加速）；升级要注意什么（第三方库兼容 + 行为差异）。</div>
  </div>
</div>

---

# 4.10 ZGC 是什么？为什么停顿低？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念：</b>JDK 11 推出的低延迟收集器，目标：停顿 ≤10ms、停顿<b>不随堆大小增长</b>、支持 8MB~4TB 堆。几乎所有暂停只依赖 <b>GC Roots 集合大小</b>——把大部分标记、重定位放到并发阶段，STW 只保留很短的根扫描。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">两大关键技术：</b>① <b>着色指针（Colored Pointers）</b>：指针里带状态位，GC 快速判断对象状态；② <b>读屏障（Load Barrier）</b>：线程读对象时若对象已搬迁，屏障把引用"修正到新地址"（自愈跳转）——对象迁移和业务线程<b>并发</b>进行。对比 G1 的转移阶段完全 STW 且停顿随存活对象增长。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">适用与代价：</b>适合延迟敏感服务（交易链路、实时推荐、网关核心路径）；代价：吞吐略低、CPU 开销更高、调优观测更复杂——工程上"先 G1，延迟 SLA 很严再上 ZGC"。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>读屏障为什么能自愈（修正到新地址后业务线程继续）；ZGC 和 G1 停顿差异本质（并发转移 vs STW 转移）；着色指针怎么区分对象状态（指针高位存标记位）。</div>
  </div>
</div>

---

# 4.11 G1 触发 Full GC 的典型场景与排查？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">to-space exhausted</b><br>复制存活对象时没有足够可用 Region（evacuation failure）。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">老年代增长太快</b><br>并发标记没完成就快打满（回收跟不上分配）。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">Humongous 大对象</b><br>超过 Region 一半的对象申请失败或碎片严重。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">Metaspace 压力</b><br>元空间过大触发。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">优化方向：</b>先看 Full GC 前后<b>老年代占用曲线和分配速率</b>（不只看停顿时间）；加堆/预留空间（<code>-Xmx</code>、<code>G1ReservePercent</code>）；避免大对象（必要时调 <code>G1HeapRegionSize</code>）；限制显式 GC（<code>DisableExplicitGC</code>）；查内存泄漏和晋升压力。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>To-space exhausted 和晋升失败的关系（复制没空间）；大对象为什么触发 Full GC（Humongous 直接进老年代占大块）；DisableExplicitGC 有风险吗（System.gc 失效，RMI 等依赖的要小心）。</div>
  </div>
</div>

---

# 4.12 GC 日志快速看什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（30 秒五看点）：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">① GC 类型</b><br>Young / Mixed / Full。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">② 停顿时长</b><br>是否超过 SLA（如 >200ms）。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">③ 回收效果</b><br>before → after 降了多少。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">④ 触发原因</b><br>Allocation Failure / Humongous / Metadata / System.gc。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">⑤ 老年代趋势</b><br>Old 是否持续上涨（晋升压力/泄漏风险）。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">正常 vs 故障示例：</b>① 正常 Young GC：<code>Pause Young ... 1536M-&gt;1180M 6.1ms</code>——类型 Young、6.1ms 可接受、回收 356M；但 Old 210→228 说明有晋升，需观察。② 故障信号：<code>To-space exhausted</code> → <code>Pause Full 7.8G-&gt;6.9G 1.42s</code>——复制没空间触发 Full GC、1.42s 重停顿、回收后空间仍紧——加堆/提前并发标记/查大对象和泄漏。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试一句话：</b>"先看类型和时长，再看回收前后堆占用和触发原因，最后看老年代趋势；<code>To-space exhausted / Evacuation Failure / Pause Full</code> 是高风险信号。"</div>
  </div>
</div>

---

# 4.13 STW 能完全避免吗？G1 为什么必须 STW？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">不能完全避免。</b>即使是低停顿收集器，也有根扫描、重标记等阶段需要短暂停顿。工程目标不是"零 STW"，而是把停顿压到业务可接受范围并保持可预测。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">G1 哪些阶段必须 STW：</b>① <b>根对象快照（Root Scan）</b>；② <b>Young/Mixed 回收中的对象复制（Evacuation Pause）</b>；③ 引用关系修正的关键切换点（remark/cleanup 的一部分）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">为什么必须停：</b>GC 线程搬对象、改引用，业务线程同时读写对象——不停顿做关键切换会出现"对象被搬走但引用还没统一修正"的不一致。STW 的作用是短时间内拿到安全一致的内存视图。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">G1 的目标：</b>不是"无停顿"而是"<b>可控停顿</b>"——通过 Region 回收和回收集预测把每次停顿控制在目标内（MaxGCPauseMillis），而不是像传统 Full GC 一次长暂停。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>ZGC 怎么做到几乎无 STW（并发转移 + 读屏障，见 4.10）；哪些收集器停顿可预测（G1/ZGC vs Parallel）。</div>
  </div>
</div>

---

# 4.14 OOM 定位的第一步是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">第一步：先判 OOM 类型</b>——不同类型抓证据和排查工具完全不同，判错方向会白费时间（把 Direct buffer 当堆泄漏看 Heap Dump 就跑偏了）。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Java heap space</b><br>堆问题 → HeapDump + GC 日志（MAT 看 Dominator Tree/引用链）。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">GC overhead limit</b><br>堆快满且回收效率极差 → 同堆链路。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Metaspace</b><br>类元数据区 → 看类加载量/ClassLoader 统计（类加载器泄漏）。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Direct buffer memory</b><br>堆外直接内存 → NMT/Netty buffer、MaxDirectMemorySize。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">unable to create native thread</b><br>线程数/系统资源 → jstack 线程数、ulimit、-Xss。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">最后归类：</b>泄漏（对象不释放）/ 突增（流量任务峰值）/ 配置不合理（堆、元空间、线程栈、直接内存参数太小）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">一句话：</b>OOM 定位不是先调参数，而是先"判类型"，再用对应证据链定位是泄漏、突增还是配置问题。（完整命令链见 4.6）</div>
  </div>
</div>

---

# 4.15 Netty 堆外内存泄漏怎么排查？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>堆外内存（off-heap）用 <code>ByteBuffer.allocateDirect()</code> 申请——网络/文件 I/O 时数据本来就会进入堆外区域，直接用堆外可减少一次从堆内到堆外的拷贝；可通过 <code>-XX:MaxDirectMemorySize</code> 限制最大堆外内存，DirectByteBuffer 失去引用后 GC 时可能回收，<b>堆外持续泄漏也会触发 OOM</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">排查流程：</b>① 先看 direct memory 指标和 Full GC 行为；② 启用泄漏检测（<code>ResourceLeakDetector</code>）定位未释放 ByteBuf；③ 排查重点是<b>引用计数对象是否成对 release</b>，以及异常分支是否遗漏释放逻辑。</div>
  </div>
  <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>线上开启最高级泄漏检测的代价是什么？</div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
