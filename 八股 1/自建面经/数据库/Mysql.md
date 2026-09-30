# MySQL

## 一、锁

# 1.1 MySQL 中有哪些锁分类？请说明全局锁、表级锁、行锁、间隙锁、意向锁等锁的作用和区别。

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">MySQL 的锁可以按<b>作用范围</b>分为全局锁、表级锁和行级锁，也可以按<b>兼容关系</b>分为共享锁（读锁）和排他锁（写锁）。</div>
    <div style="display:grid;grid-template-columns:100px 1fr;column-gap:12px;row-gap:6px;margin:8px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">全局锁</div>
      <div>锁定整个数据库实例，典型命令 <code>FLUSH TABLES WITH READ LOCK</code>，常用于一致性备份，期间通常不允许写入。</div>
      <div style="color:#3A5FBF;font-weight:600;">表级锁</div>
      <div>锁定整张表，粒度大、开销小，但并发度低；常见有表锁、元数据锁（MDL）和意向锁。</div>
      <div style="color:#3A5FBF;font-weight:600;">行级锁</div>
      <div>只锁定满足条件的记录，并发度高，但加锁开销更大；InnoDB 的记录锁、间隙锁和临键锁都属于行锁体系。</div>
      <div style="color:#3A5FBF;font-weight:600;">间隙锁</div>
      <div>锁定索引记录之间的范围，防止其他事务在范围内插入，从而避免幻读，主要出现在可重复读隔离级别下的范围查询。</div>
      <div style="color:#3A5FBF;font-weight:600;">意向锁</div>
      <div>表级别的标记锁，表示事务准备在表中的某些行上加共享锁或排他锁，方便表锁快速判断是否存在行锁冲突。</div>
    </div>
    <div style="margin:8px 0 0;">实际加什么锁还取决于存储引擎、索引是否命中、事务隔离级别和 SQL 的范围条件。排查锁等待时要结合执行计划、事务状态和锁等待信息，不能只看 SQL 表面。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>间隙锁为什么能防幻读（锁定范围阻止插入）；意向锁的作用（表锁与行锁的冲突快速判断）；记录锁/间隙锁/临键锁的区别（记录 / 区间 / 左开右闭区间）；MDL 锁是什么（表结构变更时的元数据锁）。</div>
  </div>
</div>

## 二、索引

---

# 2.1 MySQL 索引失效的常见情况有哪些？例如隐式类型转换、违反最左前缀法则、以 `%` 开头的 `LIKE` 查询等。

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：蔚来</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">常见情况包括：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">函数/表达式/计算</b><br>对索引列做函数或计算，例如 <code>DATE(create_time)</code>，无法利用原始有序值。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">隐式类型转换</b><br>字符串列用数字条件比较，导致索引列被转换。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">违反最左前缀</b><br>跳过最左列或在中间列断开，无法利用整棵联合索引。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">左模糊 LIKE</b><br><code>LIKE '%abc'</code> 无法按前缀定位；<code>'abc%'</code> 可以。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">OR 一侧无索引</b><br><code>OR</code> 时其中一侧没有索引，优化器可能放弃索引，具体看成本估算。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">!= / NOT IN / 大范围</b><br>选择性变差，优化器可能选择全表扫描。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">数据量/统计信息</b><br>数据量很小、返回比例高或统计信息不准确时，全表扫描可能更便宜。</div>
    </div>
    <div style="margin:8px 0 0;">排查时使用 <code>EXPLAIN</code> 查看 <code>key</code>、<code>type</code>、<code>rows</code> 和 <code>Extra</code>，再结合实际数据分布判断，不应把“执行计划没有使用索引”简单等同于数据库出错。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么函数会让索引失效（B+ 树存的是原始列值，无法按函数结果定位）；<code>%abc%</code> 怎么优化（覆盖索引/倒排/前缀索引权衡）；范围条件后索引失效的边界（最左前缀到范围为止）。</div>
  </div>
</div>

## 三、存储引擎与索引结构

---

# 3.1 数据库为什么使用 B+ 树而不是普通平衡树？B 树和 B+ 树有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理（为什么不用平衡树）：</b>数据库索引首先要考虑磁盘或 SSD 的<b>随机 I/O 次数</b>。普通平衡二叉树查询复杂度是 <code>O(log n)</code>，但每个节点只有两个子节点，数据量大时树高较高；如果一个节点对应一个数据页，每向下一层都可能产生一次随机 I/O。B 树和 B+ 树是<b>多路平衡查找树</b>，一个节点可保存大量键和子节点指针，分支因子更大，树高通常只有几层，更适合页式存储。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">B 树</b>
        <div style="margin:4px 0 0;">非叶子节点和叶子节点都可以保存完整记录。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>非叶子节点存数据 → 同页容纳索引项少、扇出小、树更高；查询路径不稳定（可能中途命中）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">B+ 树</b>
        <div style="margin:4px 0 0;">非叶子节点只保存索引键和子节点指针，完整记录集中在叶子节点；叶子节点间按键值顺序连接。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">优势：</b>扇出更大、层数更低；所有查询都到叶子、路径稳定；范围查询沿叶子链表顺序扫描。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">因此，B+ 树比普通平衡树更少访问数据页，比 B 树具有更高扇出和更好的范围扫描能力，更符合数据库按页读取、预读和顺序访问的特点。它不是在所有场景都优于其他结构：以写入吞吐为主的 KV 存储可能选择 LSM Tree，纯内存精确查询也可能使用哈希表。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>B+ 树和 B 树谁更适合范围查询（B+，叶子链表）；为什么路径稳定重要（I/O 可预测）；哈希索引为什么不适合范围查询（无序）；LSM 树什么时候优于 B+ 树（写密集）。</div>
  </div>
</div>

---

# 3.2 B+ 树索引是怎么更新的？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">插入流程：</b>InnoDB 从根页开始，根据索引键在内部节点中查找子页，直到定位到目标叶子页。叶子页有足够空间就直接插入合适位置；空间不足触发<b>页分裂</b>——把部分记录移动到新页，并把新的分隔键写入父节点；父节点也满时继续向上分裂，极端情况生成新的根页（树增高一层）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">删除与更新：</b>删除记录通常先做<b>删除标记</b>，后续由后台清理；页面利用率过低时发生页合并或重新组织。更新主键 = 删除旧索引记录 + 插入新记录，因此主键应尽量稳定；更新普通列时，如果该列出现在二级索引中，需要同步修改对应二级索引。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节（写入路径）：</b>这些操作不是每次都直接同步写入磁盘。目标页通常先加载到 <b>Buffer Pool</b>，在内存中修改为脏页；事务提交前按 <b>WAL</b> 原则保证相关 <code>redo log</code> 持久化，脏页再由后台线程刷盘。<code>undo log</code> 用于事务回滚和 MVCC；部分符合条件的二级索引页不在内存时，可通过 <b>Change Buffer</b> 延迟合并，减少随机读盘。随机主键容易让插入位置分散并频繁分裂，<b>递增主键更容易在叶子链表尾部顺序写入</b>，但分布式系统还要考虑热点和 ID 生成方式。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>页分裂会带来什么问题（写放大 + 空间碎片）；为什么递增主键写入快（尾部顺序写，少分裂）；Change Buffer 缓存什么（二级索引的变更）；WAL 为什么先写日志再刷页（崩溃恢复）。</div>
  </div>
</div>

---

# 3.3 InnoDB 的索引结构是什么？什么是回表和覆盖索引？如何减少磁盘读取次数？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>InnoDB 的主要索引使用 B+ 树。主键索引是<b>聚簇索引</b>（叶子节点保存完整行记录）；二级索引的叶子节点保存<b>二级索引键 + 主键值</b>。查询二级索引时，如果需要的字段不在二级索引中，就先从二级索引取到主键，再到主键 B+ 树查询完整行——这一步称为<b>回表</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">覆盖索引：</b>查询需要的列全部可以从某个索引取得时，直接返回索引内容、不需要回表。例如联合索引 <code>(user_id, status)</code>，只查这两列时直接用该索引返回。执行计划的 <code>Extra</code> 显示 <code>Using index</code>，通常说明使用了覆盖索引，但仍要结合完整执行计划判断。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">减少磁盘读取的方向：</b>① 选择区分度和顺序合理的联合索引，使过滤、排序和返回字段尽可能由一个索引完成；② 避免 <code>SELECT *</code>，只读需要的列；③ 减少大范围回表，必要时先用覆盖索引查主键再关联原表（延迟关联）；④ 保证 Buffer Pool 容量和命中率；⑤ 用 <code>EXPLAIN</code> / <code>EXPLAIN ANALYZE</code> 检查访问类型、扫描行数和实际耗时。注意：索引越多，写入维护和存储成本也越高。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么二级索引存主键而不是行地址（主键移动时二级索引不用改）；回表两次 I/O 怎么优化（覆盖索引）；延迟关联是什么（先索引查主键再 JOIN 原表）。</div>
  </div>
</div>

---

# 3.4 聚簇索引和非聚簇索引有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">聚簇索引</b>
        <div style="margin:4px 0 0;">叶子节点直接保存完整行，主键查询只需沿一棵 B+ 树定位。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">规则：</b>一张表只能有一个——优先主键；没有主键选合适的非空唯一索引；都没有则生成内部隐藏标识。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>主键值出现在每个二级索引叶子节点，主键过长会放大所有二级索引空间占用；主键变更导致行移动。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">非聚簇索引（二级索引）</b>
        <div style="margin:4px 0 0;">叶子节点保存索引列 + 聚簇索引键，不保存完整行。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">查询：</b>查其他列需按主键回表；所需字段都在索引中则覆盖索引免回表。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">数量：</b>一张表可以建多个二级索引。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">主键值会出现在每个二级索引叶子节点中，所以 InnoDB 主键不宜过长，否则会放大所有二级索引的空间占用。<b>递增、短小、稳定</b>的主键通常有利于减少页分裂和索引体积，但具体选择还要结合分布式 ID、写入热点和业务唯一性。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么 InnoDB 必须有聚簇索引（行数据的物理组织形式）；主键为什么推荐自增（顺序写 + 二级索引体积）；没有主键会怎样（隐藏 rowid 索引）。</div>
  </div>
</div>

---

# 3.5 InnoDB 和 MyISAM 有什么区别？为什么通常选择 InnoDB？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">InnoDB</b>
        <div style="margin:4px 0 0;">支持事务和 ACID，通过 <code>redo log</code>、<code>undo log</code>、锁和 MVCC 实现崩溃恢复、回滚和并发控制。</div>
        <div style="margin:4px 0 0;">支持<b>行级锁</b>、外键和一致性读；使用聚簇索引（主键叶子存完整行）。适合订单、账户、库存等需要并发写入和数据可靠性的业务。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MyISAM</b>
        <div style="margin:4px 0 0;">不支持事务和 MVCC，主要使用<b>表级锁</b>，写操作容易阻塞整张表；异常宕机后恢复能力弱于 InnoDB。</div>
        <div style="margin:4px 0 0;">索引和数据分离存储，索引叶子保存记录地址；表行数可从元数据直接获得，但这不代表复杂条件统计一定更快。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">现代通用业务通常优先选择 InnoDB，因为它在<b>事务、并发、故障恢复和数据完整性</b>方面更适合线上系统。只有在明确不需要事务、主要只读，并经过实际测试确认收益的特殊场景，才考虑其他引擎；不能只依据“某种引擎查询快”做选择。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>MyISAM 的 COUNT(*) 为什么快（元数据直接存行数）；表级锁和行级锁的并发差异；外键只有 InnoDB 支持吗（是）。</div>
  </div>
</div>

---

# 3.6 MySQL 索引叶子节点和非叶子节点的存储空间大小一样吗？如何估算？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>在 InnoDB 中，叶子节点和非叶子节点通常都以<b>页</b>为基本存储单位（默认 16KB，取决于表空间和版本配置）——所以不能简单说“叶子节点比非叶子节点大”。差别在<b>单条记录大小</b>：非叶子页主要放分隔键和子页指针（单条目录项小），聚簇索引叶子页放完整行记录（单条可能很大），二级索引叶子页放索引列 + 主键。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">估算方式：</b>分支因子 ≈ <code>页可用空间 ÷（键大小 + 子页指针大小 + 页内开销）</code>；叶子页容量 ≈ <code>页可用空间 ÷（记录大小 + 记录头和槽位开销）</code>。叶子页通常还要预留页目录、记录头、空闲空间和填充比例，所以只能得到<b>数量级估计</b>，不能替代实际执行计划和表空间统计。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>索引键越长、行越宽，单页容纳的项越少，树高和 I/O 可能增加。设计索引应控制主键和联合索引长度，避免把大文本字段直接放入普通 B+ 树索引；需要覆盖查询时只覆盖必要列。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>16KB 页能放多少行（看行大小，估算题）；树高一般几层（千万级 3-4 层）；为什么叶子页要预留空间（防频繁页分裂）。</div>
  </div>
</div>

---

# 3.7 MySQL 索引通常存储在硬盘还是内存中？扫描索引时每次都会读取磁盘吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>索引首先<b>持久化在磁盘/SSD 表空间</b>中，重启后仍然存在；运行时 InnoDB 把索引页和数据页加载到 <b>Buffer Pool</b>，最近访问的页留在内存。查询是否产生磁盘 I/O 取决于目标页是否在 Buffer Pool、操作系统缓存、预读策略和扫描范围——不是“每次使用索引都读一次磁盘”。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">关键细节：</b>冷启动或内存不足时，索引页不在缓存，需要从存储设备读入；连续范围扫描可能触发<b>预读</b>，减少单页随机读取；即使使用索引，如果需要回表且访问记录分散，也可能产生大量数据页 I/O；覆盖索引只扫描索引页，通常更省 I/O，但索引本身也占用缓存空间。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>Buffer Pool 太小 → 命中率低 → 频繁磁盘读；索引太多 → 缓存空间被索引占满 → 数据页被挤出。排查时结合 <code>EXPLAIN</code>、<code>EXPLAIN ANALYZE</code>、Buffer Pool 命中率、磁盘 I/O、读取请求和查询延迟判断，不能因为执行计划显示 <code>ref</code> 或 <code>range</code> 就断言一定很快——还要看扫描页数、回表比例、缓存冷热和并发竞争。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>Buffer Pool 命中率怎么查（<code>SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read%'</code>）；预读是什么（顺序扫描时提前读下一批页）；索引和数据都放内存会怎样（内存不够，缓存竞争）。</div>
  </div>
</div>

---

# 3.8 MySQL Spider 了解吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>Spider 是面向分布式或分片场景的<b>存储引擎方案</b>：把一张逻辑表的数据映射到多个远程 MySQL 或兼容节点，对上层尽量提供统一的 SQL 访问方式。查询进入 Spider 表后，存储引擎根据分区或路由规则把请求<b>下推</b>到远程节点，再汇总结果。它不是 InnoDB 的替代品——远端实际数据通常仍由 InnoDB 等引擎保存。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">优势与代价：</b><b>优势</b>：对应用暴露相对统一的表结构，减少应用直接维护多数据源路由的复杂度。<b>代价</b>：跨节点查询、排序、聚合和事务受网络、下推能力和节点状态影响，执行计划和故障排查更复杂。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>使用前重点评估：分片键、跨分片查询比例、远程连接池、超时重试、节点故障和数据一致性。面试中要说明 Spider 并不是 MySQL 默认内置且普遍使用的分库分表方案，不应和 MySQL Router、读写分离代理或 ShardingSphere 混为一谈——是否采用要看实际发行版、运维能力和业务访问模式。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>Spider 和 ShardingSphere 的区别（存储引擎层 vs 中间件层）；下推是什么（远程节点执行再汇总）；跨分片事务怎么做（不推荐，收敛到单分片）。</div>
  </div>
</div>

---

# 3.9 了解 LSM 树吗？它和 B+ 树有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>LSM Tree 的核心思想是<b>把随机写转换为顺序写</b>。写请求先记录 WAL，再写入内存中的 MemTable；MemTable 达到阈值后转为不可变结构并顺序刷成磁盘上的 SSTable；后台通过 Compaction 合并不同层级的 SSTable，清理被覆盖或删除的数据。读取时可能需要检查 MemTable、多个 SSTable 和布隆过滤器，因此需要缓存、索引和 Compaction 控制读放大。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">B+ 树</b>
        <div style="margin:4px 0 0;">直接按键定位并修改数据页，点查和范围查询路径稳定，适合读写均衡和关系数据库场景。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>随机写可能引起页分裂和离散 I/O。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">LSM Tree</b>
        <div style="margin:4px 0 0;">写入吞吐通常更高，适合日志、时序数据和写密集型 KV 存储。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>读放大、写放大、空间放大以及 Compaction 对 I/O 的影响。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">两者没有绝对优劣。选择时要看<b>读写比例、点查与范围查询、磁盘类型、延迟目标、数据量和运维复杂度</b>。MySQL InnoDB 主要使用 B+ 树；RocksDB、LevelDB 等典型 KV 引擎主要采用 LSM Tree。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>LSM 写放大是什么（一份数据多次写：WAL + MemTable + Compaction）；读放大怎么缓解（布隆过滤器 + 缓存 + 分层）；为什么时序场景适合 LSM（顺序写 + 追加式）。</div>
  </div>
</div>

---

# 3.10 RocksDB 了解吗？它和 MySQL 的存储方式有什么不同？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">RocksDB</b>
        <div style="margin:4px 0 0;">嵌入式持久化 KV 引擎，应用通过库接口在本地进程中读写键值，不像 MySQL 天然提供独立数据库服务、SQL、关系模型和完整查询优化器。</div>
        <div style="margin:4px 0 0;">主要采用 LSM Tree：写入先进 WAL 和 MemTable，再刷成 SSTable，后台 Compaction 合并文件，对高吞吐顺序写和大规模 KV 数据友好。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>应用需自己设计键编码、二级索引、数据模型、备份和服务化能力，关注 Compaction、写放大、Block Cache 和 SSTable 数量。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL（InnoDB）</b>
        <div style="margin:4px 0 0;">关系数据库：表、行、SQL、事务、二级索引、Join、权限和客户端连接协议；InnoDB 用 B+ 树组织索引页，Buffer Pool、Redo、Undo 和 MVCC 保证事务与恢复。</div>
        <div style="margin:4px 0 0;">适合复杂条件查询、关系建模和强事务业务；维护关系约束和多个索引会增加写入成本。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">RocksDB 更适合作为<b>本地状态存储、存储系统底层引擎、流处理状态后端或写密集型 KV 场景</b>；MySQL 更适合作为业务事实数据库。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>RocksDB 为什么适合做流处理状态后端（本地嵌入式 + 高写吞吐）；RocksDB 有 SQL 吗（没有，要自己建模）；什么场景选 RocksDB 而不是 MySQL（写密集 KV、嵌入场景）。</div>
  </div>
</div>

---

# 3.11 为什么 OLTP 常用 B+Tree 而不是 Hash？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">场景：</b>OLTP 典型查询不只是 <code>=</code>，还有范围查询（<code>between</code>）、排序分页（<code>order by limit</code>）、联合索引前缀查询（<code>where a=? and b&gt;?</code>）。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3F8C12;">B+Tree 为什么适合</b>
        <div style="margin:4px 0 0;">① 支持<b>范围和排序</b>（叶子有序 + 链表）；② 支持<b>最左前缀</b>，联合索引复用性高；③ <b>磁盘友好</b>：扇出高、树高 2-4 层、I/O 稳定；④ 一套索引覆盖点查/范围/排序/分页。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#B26E00;">Hash 为什么不作主索引</b>
        <div style="margin:4px 0 0;">只擅长等值查找，不支持范围/排序；无法利用最左前缀；查询能力单一，不匹配 OLTP 复杂多样的 SQL 形态。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">一句话：</b>OLTP 要的是"一套索引覆盖多种查询"——B+Tree 既支持等值也支持范围/排序且 I/O 稳定；Hash 点查快但能力太窄。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>Hash 索引在哪用（内存表/精确匹配场景）；为什么不直接用 Hash + 链表（范围查询无解）。</div>
  </div>
</div>

## 四、索引与查询优化

---

# 4.1 创建 MySQL 联合索引需要注意什么？最左匹配原则如何理解？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理（最左匹配）：</b>联合索引按索引列从左到右的顺序组织 B+ 树，例如索引 <code>(a, b, c)</code> 首先按 <code>a</code> 排序，<code>a</code> 相同时再按 <code>b</code> 排序，<code>a、b</code> 都相同时再按 <code>c</code> 排序。因此查询通常要从最左列开始连续使用：条件只有 <code>b</code> 或 <code>c</code> 时无法直接利用整棵索引；条件包含 <code>a</code>，或 <code>a、b</code>，才符合最左前缀。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">设计注意：</b>① 结合真实查询设计，而不是机械地把区分度最高的字段放最前；② 优先覆盖高频等值过滤、租户或业务范围、排序和分组字段，再考虑范围条件和需要覆盖的返回列；③ <b>范围条件之后的列通常不能继续缩小扫描区间</b>，但有时仍可通过 Index Condition Pushdown 在索引层过滤；④ 避免与现有索引前缀重复的冗余索引——已有 <code>(a, b)</code> 时，单列索引 <code>(a)</code> 可能没必要。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>索引列不是越多越好——联合索引增加磁盘占用、Buffer Pool 压力和插入更新成本；字段过长降低单页容纳的索引项数量。最终用 <code>EXPLAIN</code>、实际数据分布和线上慢查询验证，关注是否减少扫描行数、排序、临时表和回表，不能只看 <code>key</code> 字段是否出现索引名。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>范围条件后为什么索引失效（排序只保证同 a、b 内 c 有序）；ICP 是什么（索引条件下推，索引层过滤减少回表）；<code>(a,b)</code> 索引能覆盖 <code>a</code> 单列查询吗（能，最左前缀）。</div>
  </div>
</div>

---

# 4.2 对字段 A、B、C 建立联合索引，查询条件为 `A = 1 AND B > 2 AND C = 3`，哪些条件可以使用索引？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">结论：</b>对于联合索引 <code>(A, B, C)</code>，<code>A = 1</code> 可以使用索引精确定位；接着 <code>B &gt; 2</code> 可以继续确定扫描区间；由于 <code>B</code> 是范围条件，<code>C</code> 在 B+ 树排序中只保证在相同 <code>A、B</code> 值内部有序，因此<b>通常不能继续用于缩小连续的索引扫描范围</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">C 条件并非完全没用：</b>如果 MySQL 使用 <b>Index Condition Pushdown（ICP）</b>，且 <code>C</code> 已包含在联合索引中，存储引擎可以在索引层先判断 <code>C = 3</code>，减少回表数量——执行计划的 <code>Extra</code> 可能显示 <code>Using index condition</code>；如果查询字段也都被索引覆盖，还可能直接使用覆盖索引。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节（面试表述）：</b>“用于索引定位和确定范围的是 A、B，C 通常不能继续缩小范围，但可能通过索引条件下推参与过滤。”实际情况仍需结合 MySQL 版本、优化器选择和 <code>EXPLAIN ANALYZE</code> 结果确认。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>如果把条件改成 <code>B &gt; 2 AND C = 3</code> 呢（没有 A，直接无法使用该索引）；范围条件放最后是不是更好（是，联合索引设计原则）；ICP 和覆盖索引的区别（索引层过滤 vs 免回表）。</div>
  </div>
</div>

---

# 4.3 两千万数据量的 MySQL 表应该如何设计和优化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>两千万行本身并不意味着必须分库分表，首先要看单行大小、访问模式、并发量、增长速度和延迟目标。优化顺序：表结构 → 查询 → 架构，逐层看瓶颈。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">表结构层：</b>选择合适的数据类型，避免无意义的大字段和过长主键；主键尽量短小、稳定；常用过滤和排序条件建立合理索引；大文本、低频字段按访问模式拆到扩展表，但不能为了“看起来规范”盲目拆表。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">查询优化层：</b>从慢日志和执行计划出发，减少全表扫描、深分页、返回大字段和无效回表。列表查询用覆盖索引和基于主键或时间的<b>游标分页</b>；历史数据归档到历史表或冷存储；按时间范围访问且有明确生命周期时考虑<b>分区</b>——分区键必须出现在高频查询条件中，分区也不能替代索引。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">架构层：</b>Redis 缓存热点数据、只读副本分担查询、异步化非核心操作、搜索引擎承接复杂检索。只有单实例容量、写入吞吐或维护窗口确实成为瓶颈时，才考虑<b>分库分表</b>，并提前设计分片键、全局 ID、跨分片查询、扩容和数据迁移。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>分库分表引入的复杂度（跨片查询、全局 ID、扩容）远大于收益时不要提前做；拆表/分区要评估查询模式变化；优化后必须用真实 SQL 和数据量验证 P95/P99 延迟、扫描行数、Buffer Pool 命中率、磁盘 I/O 和主从延迟。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>什么时候才必须分库分表（容量/写入吞吐/维护窗口是硬瓶颈）；分区和分表的区别（单机内 vs 跨实例）；深分页怎么优化（游标分页/延迟关联）。</div>
  </div>
</div>

---

# 4.4 MySQL 慢查询如何优化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>慢查询优化是<b>定位 → 优化 → 验证</b>三步循环，第一步是定位而不是直接加索引。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">第一步定位：</b>开启并分析慢查询日志，结合监控确定慢 SQL 的频率、P95/P99、扫描行数、锁等待和业务影响；再用 <code>EXPLAIN</code> / <code>EXPLAIN ANALYZE</code> 查看访问顺序、索引、估算与实际行数、回表、排序和临时表。估算行数与实际差异大时，检查统计信息和数据倾斜。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">第二步优化 SQL 和索引：</b>只查询必要列；避免在索引列上做函数或隐式转换；减少不必要的子查询、<code>OR</code>、大范围扫描和深分页；按过滤、关联、排序和返回字段设计联合索引，尽量用覆盖索引；Join 检查连接字段类型和字符集一致，小结果集尽早过滤，避免循环逐条查询造成 N+1。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">第三步看系统和架构：</b>SQL 已合理但仍慢时，检查 Buffer Pool、磁盘 I/O、CPU、连接池、锁等待、长事务和主从延迟；热点查询可以缓存，报表或复杂搜索异步化或交给分析系统。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>每次改动都要在接近真实的数据量上对比执行计划、扫描行数、耗时和资源，防止为一个 SQL 增加索引却拖慢整张表写入；索引不是越多越好。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>EXPLAIN 和 EXPLAIN ANALYZE 的区别（估算 vs 实际执行）；慢查询日志怎么开（long_query_time 阈值）；N+1 查询是什么（循环中逐条查）。</div>
  </div>
</div>

---

# 4.5 `EXPLAIN` 应该重点关注哪些字段？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">重点关注字段：</div>
    <div style="display:grid;grid-template-columns:140px 1fr;column-gap:12px;row-gap:6px;margin:8px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">id / select_type</div>
      <div>判断查询块、子查询和 UNION 的执行结构。</div>
      <div style="color:#3A5FBF;font-weight:600;">table / partitions</div>
      <div>当前访问的表、派生表以及命中的分区。</div>
      <div style="color:#3A5FBF;font-weight:600;">type</div>
      <div>访问方式，性能从好到差大致为 <code>system</code>、<code>const</code>、<code>eq_ref</code>、<code>ref</code>、<code>range</code>、<code>index</code>、<code>ALL</code>。出现 <code>ALL</code> 不一定错误——小表全扫可能更便宜，但大表要重点关注。</div>
      <div style="color:#3A5FBF;font-weight:600;">possible_keys / key / key_len</div>
      <div>候选索引、实际选择的索引以及使用的索引长度，可辅助判断联合索引使用到哪些部分。</div>
      <div style="color:#3A5FBF;font-weight:600;">ref</div>
      <div>索引列与常量或其他表字段如何比较。</div>
      <div style="color:#3A5FBF;font-weight:600;">rows / filtered</div>
      <div>预计扫描行数和条件过滤比例，两者结合可估算传给下一步的数据量。</div>
      <div style="color:#3A5FBF;font-weight:600;">Extra</div>
      <div>关注 <code>Using index</code>（覆盖索引）、<code>Using index condition</code>（ICP）、<code>Using where</code>、<code>Using temporary</code>、<code>Using filesort</code> 等信息。</div>
    </div>
    <div style="margin:8px 0 0;"><code>EXPLAIN</code> 主要是<b>优化器估算</b>，不等于真实执行情况。条件允许时用 <code>EXPLAIN ANALYZE</code> 对比估算行数、实际行数、循环次数和耗时；估算偏差大可能是统计信息过旧、字段相关性强或数据分布倾斜。不能只看到“使用了索引”就认为查询已经优化，还要关注实际扫描、回表和结果集大小。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>type 从 ref 退化成 ALL 说明什么（索引没被用上）；Using filesort 一定慢吗（是，需要额外排序）；key_len 能看出什么（联合索引用了几列）。</div>
  </div>
</div>

---

# 4.6 时间戳函数为什么可能导致索引失效？如何改写？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>普通 B+ 树索引保存的是<b>原始列值</b>。如果在索引列上使用函数，例如 <code>DATE(create_time) = '2026-09-07'</code>，数据库通常需要先对每行的 <code>create_time</code> 计算 <code>DATE()</code> 后再比较，无法直接根据原始有序值确定连续范围，因此可能放弃普通索引或扫描大量索引项。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">改写方式：</b>把函数计算转移到<b>常量侧</b>，改写为左闭右开的范围条件（既能用索引，也避免时间精度边界遗漏）：</div>
  </div>
</div>

```sql
-- 错误：索引列上套函数，索引失效
SELECT * FROM t WHERE DATE(create_time) = '2026-09-07';

-- 正确：函数移到常量侧，范围条件走索引
SELECT * FROM t
WHERE create_time >= '2026-09-07 00:00:00'
  AND create_time <  '2026-09-08 00:00:00';
```

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <div style="margin:0;"><b style="color:#3A5FBF;">关键细节：</b>注意字段类型和参数类型一致，防止隐式转换；确实需要按表达式高频查询时，可评估<b>生成列</b>或函数索引（MySQL 8），但会增加存储和写入维护成本，仍需执行计划验证。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么右开（避免包含次日 00:00:00）；函数索引是什么（索引存函数结果）；隐式转换为什么失效（字符串列被转成数字比较）。</div>
  </div>
</div>

---

# 4.7 SQL 分页怎么写？深分页有什么问题？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理（深分页问题）：</b>普通分页 <code>LIMIT offset, size</code>（如 <code>LIMIT 20, 10</code>）在偏移量很大时，MySQL 仍需要找到并扫描前面的记录再丢弃绝大部分；如果还要回表或排序，深分页的 I/O 和 CPU 成本明显增加，而且并发插入、删除可能导致翻页重复或遗漏。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">游标分页（Seek Method，推荐）：</b>记录上一页最后一条数据的有序键，下一页用 <code>WHERE id &gt; last_id ORDER BY id LIMIT 10</code>。按时间和主键联合排序时，用 <code>(create_time, id)</code> 作为稳定游标，条件同时比较时间和 ID。不适合直接跳到任意页，但适合连续向后翻页和滚动加载。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">深分页的其他方案：</b>① <b>延迟关联</b>：先通过覆盖索引只查询目标主键，再与原表关联获取完整数据；② 限制最大页数、提供条件筛选；③ 搜索引擎承接。无论哪种方式，<b>排序字段必须稳定且最好唯一</b>，否则分页结果可能不确定。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么 LIMIT 100000, 10 慢（扫描 100010 行再丢 100000 行）；游标分页为什么不能跳页（没有偏移概念）；排序字段不稳定会怎样（翻页重复/漏数据）。</div>
  </div>
</div>

## 五、事务与一致性

---

# 5.1 MySQL 事务有哪些特性？ACID 分别如何实现？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:grid;grid-template-columns:80px 1fr;column-gap:12px;row-gap:6px;margin:6px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">原子性</div>
      <div>事务中的多个操作要么全部成功，要么全部回滚。InnoDB 通过 <code>undo log</code> 保存修改前的信息，发生异常或显式回滚时恢复旧值。</div>
      <div style="color:#3A5FBF;font-weight:600;">一致性</div>
      <div>事务执行前后都要满足约束和业务规则（主键、唯一键、外键、检查约束、库存不为负）。数据库的锁、日志和隔离机制提供基础保障，业务代码仍要负责跨服务规则和幂等。</div>
      <div style="color:#3A5FBF;font-weight:600;">隔离性</div>
      <div>并发事务互相尽量不可见。InnoDB 通过锁、MVCC 和事务隔离级别控制读写冲突，避免脏读、不可重复读和幻读。</div>
      <div style="color:#3A5FBF;font-weight:600;">持久性</div>
      <div>事务提交成功后，即使宕机已提交结果也不能丢失。InnoDB 先把必要的 <code>redo log</code> 刷到持久化介质再认为提交完成，重启后通过日志恢复脏页。</div>
    </div>
    <div style="margin:8px 0 0;">在复制场景中还要考虑 <code>binlog</code>：MySQL 通过内部<b>两阶段提交</b>协调 InnoDB 的 <code>redo log</code> 和 Server 层的 <code>binlog</code>，避免出现存储引擎已提交但复制日志没记录，或日志已记录但引擎没提交的不一致。ACID 是数据库能力和业务约束共同实现的，不是只依赖某一个日志文件。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>redo log 和 undo log 分别保证什么（持久性 vs 原子性）；两阶段提交协调的是什么（redo 和 binlog）；一致性是数据库单独保证的吗（不是，业务也要参与）；ACID 里哪个最难实现（隔离性，锁和 MVCC 的权衡）。</div>
  </div>
</div>

---

# 5.2 事务隔离性会导致哪些问题？不可重复读和幻读有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#B26E00;">脏读</b>
        <div style="margin:4px 0 0;">一个事务读到另一个事务<b>尚未提交</b>的数据，后者回滚后前者读到的内容就不存在了。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#B26E00;">不可重复读</b>
        <div style="margin:4px 0 0;">同一事务两次读取<b>同一行</b>，期间另一事务提交了更新或删除，导致两次结果不同。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#B26E00;">幻读</b>
        <div style="margin:4px 0 0;">同一事务按<b>范围查询</b>两次，期间另一事务插入或删除了符合条件的行，导致结果集行数变化。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>三者关注点不同：脏读关注<b>未提交数据</b>，不可重复读关注<b>同一行的已提交版本变化</b>，幻读关注<b>范围内新增或消失的记录</b>。隔离级别越高问题越少，但锁等待、并发度或实现成本越高。InnoDB 的普通一致性读主要依赖 MVCC，锁定读和当前读则通过记录锁、间隙锁和临键锁阻止并发修改或插入。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>不可重复读和幻读的核心区别（行内容变化 vs 行数变化）；可重复读下还有幻读吗（快照读没有，当前读靠临键锁）；脏读为什么最严重（读到不存在的数据）。</div>
  </div>
</div>

---

# 5.3 MySQL 默认隔离级别是什么？可重复读解决幻读了吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">默认隔离级别：</b>MySQL InnoDB 默认是<b>可重复读（Repeatable Read，RR）</b>。在一个事务第一次执行普通 <code>SELECT</code> 时会形成 Read View，后续普通快照读按同一视图判断记录可见性，因此即使其他事务提交了更新，当前事务仍能看到一致的历史版本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">幻读问题（分场景）：</b>不能用一句“解决了”概括。① <b>普通快照读</b>：依靠 MVCC 读取同一快照，结果集通常保持一致，传统意义上的幻读不会出现；② <b>当前读</b>（<code>SELECT ... FOR UPDATE</code>、<code>UPDATE</code>、<code>DELETE</code>）：需要看到最新已提交数据，范围查询通过<b>记录锁、间隙锁或临键锁</b>阻止其他事务插入符合条件的新记录；③ 没有合适索引时，加锁范围和扫描范围可能扩大，锁竞争更严重。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">隔离级别选择：</b>业务只需要较高并发的已提交数据时评估<b>读已提交（RC）</b>；需要跨语句稳定快照用 RR。选择时结合读写比例、锁等待、长事务、主从延迟和业务对一致性的要求，不能只追求最高隔离级别。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>RR 和 RC 的 Read View 创建时机（事务第一次读 vs 每次读）；为什么 MySQL 选 RR 做默认（历史原因 + binlog 复制兼容）；临键锁是什么（记录锁 + 间隙锁组合）。</div>
  </div>
</div>

---

# 5.4 MVCC 是什么？它如何实现一致性读？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>MVCC（Multi-Version Concurrency Control）多版本并发控制。InnoDB 更新记录时不会立即丢掉旧值，而是通过<b>隐藏的事务信息和 <code>undo log</code> 保存版本链</b>——记录包含创建或最后修改它的事务 ID，以及指向旧版本的回滚指针；事务读取时根据自己的 Read View 沿版本链寻找可见版本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">Read View 机制：</b>Read View 是某个时刻的事务可见性快照，核心信息包括创建视图时仍活跃的事务、低水位和高水位等。对一条记录：修改它的事务已在视图创建前提交 → 该版本可见；事务未提交或在视图创建后才提交 → 回溯 <code>undo log</code> 查找更早的可见版本。这样普通 <code>SELECT</code> 不必给记录加排他锁，<b>读写可以并行</b>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>MVCC 主要服务于<b>普通一致性读</b>，不意味着所有语句都不加锁。<code>SELECT ... FOR UPDATE</code>、更新和删除属于<b>当前读</b>，需要读取最新版本并加锁。长事务会让旧版本不能及时清理，导致 <code>undo log</code> 膨胀、历史链变长和 purge 延迟——要避免长时间不提交或不结束的事务。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>快照读和当前读的区别（版本链 vs 最新+锁）；RR 和 RC 的 Read View 差异（一次 vs 每次）；undo log 为什么不能一直删（活跃事务还要看旧版本）；MVCC 能防幻读吗（快照读可以，当前读靠锁）；长事务为什么危险（undo 膨胀 + 历史链变长）。</div>
  </div>
</div>

---

# 5.5 MySQL 隔离级别中的读未提交和读已提交有什么特点？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">读未提交（RU）</b>
        <div style="margin:4px 0 0;">允许事务读取其他事务<b>尚未提交</b>的修改，可能发生脏读、不可重复读和幻读。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">适用：</b>通常拥有更高并发度和更少读阻塞，但读到的数据可能随后回滚——只有对短暂不一致完全不敏感的统计或监控场景才可能考虑，核心交易不使用。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">读已提交（RC）</b>
        <div style="margin:4px 0 0;">只读取已经提交的数据，避免脏读；但同一事务中两次普通查询可能创建<b>不同的 Read View</b>，仍可能出现不可重复读和幻读。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">特点：</b>比 RR 更容易看到最新提交结果，锁定范围可能更小；业务需要跨多条语句稳定快照时，要评估额外的锁、版本或应用层一致性方案。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">在 InnoDB 中，普通一致性读主要通过 MVCC 实现，更新、删除和 <code>SELECT ... FOR UPDATE</code> 等当前读仍会加锁。隔离级别选择要结合业务语义、长事务、锁等待和副本延迟，不能只按照“隔离级别越低性能越高”做判断。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>RC 和 RR 哪个锁范围小（RC，间隙锁只在 RR 生效）；为什么 RC 还有不可重复读（每次读新建 Read View）；哪些场景用 RU（几乎不用，统计可容忍脏读）。</div>
  </div>
</div>

---

# 5.6 MySQL 除了 `redo log` 还有哪些日志？`redo log` 中保存什么内容？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:grid;grid-template-columns:110px 1fr;column-gap:12px;row-gap:6px;margin:6px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">redo log</div>
      <div>InnoDB 层的物理或偏物理恢复日志，记录数据页发生的修改及恢复所需信息。固定大小循环文件，用于 <b>WAL 和崩溃恢复</b>——解决已提交事务的数据页还没来得及刷盘时如何恢复。</div>
      <div style="color:#3A5FBF;font-weight:600;">undo log</div>
      <div>保存修改前的逻辑信息，用于事务回滚和 MVCC 读取历史版本；清理受活跃事务影响。</div>
      <div style="color:#3A5FBF;font-weight:600;">binlog</div>
      <div>Server 层的二进制归档日志，按事件记录提交后的数据变更（statement/row/mixed 格式），用于主从复制、数据恢复和 CDC。</div>
      <div style="color:#3A5FBF;font-weight:600;">慢查询日志</div>
      <div>记录超过阈值或未使用索引等慢 SQL，用于性能分析；通用查询日志更完整但开销更高。</div>
      <div style="color:#3A5FBF;font-weight:600;">错误/中继日志</div>
      <div>错误日志记录启动、停止、崩溃、恢复和关键错误；中继日志由复制从库接收主库 binlog 后保存，用于后续重放。</div>
    </div>
    <div style="margin:8px 0 0;"><code>redo log</code> 保存的不是完整 SQL 文本，而是<b>存储引擎层面用于重做页面修改的信息</b>；<code>binlog</code> 也不是用来替代 <code>undo log</code> 的。事务提交时要协调 <code>redo log</code> 和 <code>binlog</code>，恢复和复制分别使用不同日志完成各自职责。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>redo log 和 binlog 谁先写（两阶段提交）；为什么 redo 是物理日志（记录页修改，恢复快）；binlog 能用于崩溃恢复吗（不能，是逻辑复制日志）；redo 和 undo 的分工（redo 管已提交不丢=持久性，undo 管未提交回滚=原子性 + 历史版本读=隔离性）。</div>
  </div>
</div>

---

# 5.7 为什么有 `binlog` 还需要 `redo log`？两者分别解决什么问题？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">redo log</b>
        <div style="margin:4px 0 0;">InnoDB 存储引擎层，面向<b>数据页修改</b>，循环写入。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">职责：</b>WAL 和崩溃恢复——事务提交时脏页没刷回表空间也没关系，redo log 已持久化，重启后把修改重新应用到数据页。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">binlog</b>
        <div style="margin:4px 0 0;">MySQL Server 层，记录已提交的<b>逻辑变更事件</b>。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">职责：</b>主从复制、增量备份、按时间点恢复和 CDC；需要支持不同存储引擎和下游订阅者，不能承担 InnoDB 页面恢复的职责。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">单独依赖 <code>binlog</code> 做崩溃恢复，无法替代存储引擎对脏页和页结构的恢复机制。事务提交时通过<b>两阶段提交</b>协调两种日志：先准备 <code>redo log</code> → 写入并刷 <code>binlog</code> → 提交 <code>redo log</code>，这样宕机恢复时可以根据日志状态判断事务是否应该完成，避免主库数据与复制日志不一致。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>两阶段提交哪一步失败怎么恢复（按日志状态判断回滚或补齐）；redo 为什么不用 binlog 替代（层次不同：物理页 vs 逻辑事件）；binlog 什么时候刷盘（提交时，sync_binlog 控制）。</div>
  </div>
</div>

---

# 5.8 什么情况下需要使用 `binlog`？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">需要记录数据库变更并供其他系统消费时就会使用 <code>binlog</code>：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">主从复制</b><br>从库读取主库 binlog 并重放，保持副本数据一致。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">时间点恢复</b><br>全量备份 + binlog 回放，把数据库恢复到某个具体时刻。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">CDC 数据同步</b><br>订阅 row 格式 binlog，把变更发送到搜索、数仓、缓存或消息系统。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">格式选择：</b>statement 记录 SQL（日志量小但依赖执行环境和非确定函数）；row 记录行变化（复制和 CDC 更准确，日志量可能更大）；mixed 按语句类型选择。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>生产要配置日志保留周期、磁盘容量、刷盘策略、GTID、脱敏和访问权限——避免日志过期导致增量链断裂，也要防止敏感字段直接暴露给下游。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>row 和 statement 格式怎么选（准确 vs 体积）；binlog 保留多久（备份周期 + 恢复窗口）；Canal 是什么（订阅 binlog 的 CDC 工具）。</div>
  </div>
</div>

---

# 5.9 binlog 主从复制是拉还是推？redo/binlog 两阶段提交谁先提交？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：字节</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">binlog 是拉还是推：</b><b>从库主动拉取（pull）</b>——从库的 I/O 线程向主库发起请求，主库的 dump 线程把 binlog 推给从库，从库写入 relay log 后由 SQL 线程重放。不是主库主动推送：从库自己控制拉取进度（binlog 位点/GTID），断线后从上次位点续拉。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">两阶段提交顺序（先 redo prepare → binlog → redo commit）：</b>① <b>prepare 阶段</b>：事务执行完，InnoDB 写 redo log 并标记为 prepare 状态；② <b>commit 阶段</b>：Server 层写 binlog 并刷盘（<code>sync_binlog=1</code> 时）→ 然后 InnoDB 把 redo 标记为 commit，事务才算提交成功。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节（为什么这个顺序）：</b>宕机恢复时按日志状态判断：① redo prepare 且 binlog 已写入 → 事务<b>补齐提交</b>（redo commit）；② redo prepare 但 binlog 没写 → 事务<b>回滚</b>。保证<b>主库数据和复制日志一致</b>——不会出现"主库提交了但从库没日志"或"日志有但从库状态没有"。binlog 必须介于两者之间才能作为两者的"对账凭证"。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>如果 binlog 写成功但 redo commit 失败会怎样（恢复时按 binlog 存在补齐提交）；sync_binlog=0 的后果（binlog 未刷盘，宕机可能丢日志）；半同步复制是什么（主库等至少一个从库 ack 才返回提交成功）。</div>
  </div>
</div>

## 六、SQL 与查询

---

# 6.1 手写 SQL：分组 Top N / 平均分大于 80 / 每科都不低于 80

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：字节</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">思路：</b>三类经典写法——① 每科不低于 80：分组取 <code>MIN(score)</code> 用 HAVING 过滤；② 平均分：<code>AVG</code> + HAVING（聚合条件必须放 HAVING，WHERE 在分组前过滤用不了聚合函数）；③ 分组 Top N：<code>GROUP BY</code> + 窗口函数 <code>ROW_NUMBER()</code> + 外层过滤（并列排名时比 LIMIT 更严谨）。参考实现：</div>
  </div>
</div>

```sql
-- ① 每科都不低于 80 的学生（先分组取最低分再过滤）
SELECT student_id
FROM score
GROUP BY student_id
HAVING MIN(score) >= 80;

-- ② 平均分大于 80 的学生
SELECT student_id, AVG(score) AS avg_score
FROM score
GROUP BY student_id
HAVING AVG(score) > 80;

-- ③ 2025 年消费额前三的用户（user × order 两表）
SELECT user_id, total
FROM (
    SELECT u.id AS user_id,
           SUM(o.amount) AS total,
           ROW_NUMBER() OVER (ORDER BY SUM(o.amount) DESC) AS rn
    FROM user u
    JOIN orders o ON u.id = o.user_id
    WHERE YEAR(o.create_time) = 2025
    GROUP BY u.id
) t
WHERE rn <= 3;
```

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <div style="margin:0;"><b style="color:#3A5FBF;">关键细节：</b>分组 TopN 三件套 = GROUP BY + 窗口函数 + 外层过滤；<code>RANK</code> 和 <code>DENSE_RANK</code> 的区别（并列是否占位）；HAVING vs WHERE（聚合后 vs 聚合前）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>取每组前三用 ROW_NUMBER 还是 RANK（看是否允许并列占位）；DISTINCT 和 GROUP BY 去重区别；JOIN 后 GROUP BY 要注意什么（连接字段唯一性防结果放大）。</div>
  </div>
</div>

---

# 6.2 左连接和右连接的区别？两表内联查询 `on` 和 `where` 的区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：北方新宇、得物</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">LEFT JOIN</b>
        <div style="margin:4px 0 0;">以左表为基准，左表全部行保留，右表无匹配补 NULL。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">RIGHT JOIN：</b>对称——以右表为基准；实际写法上 LEFT JOIN 更通用（调整表顺序即可，RIGHT 可读性差，多数团队规范禁用）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">ON vs WHERE（内连接）</b>
        <div style="margin:4px 0 0;"><b>内连接（INNER JOIN）下二者结果等价</b>，区别在语义和执行阶段：ON 在<b>连接阶段</b>决定两表如何配对，WHERE 在<b>连接完成后</b>过滤结果行。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">外连接时差异巨大：</b><code>LEFT JOIN t2 ON t1.id = t2.id AND t2.status = 1</code> 只影响匹配条件（不匹配仍保留左表行，status 为 NULL）；<code>ON t1.id = t2.id WHERE t2.status = 1</code> 会过滤掉无匹配行（等效内连接）。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节：</b>左连接想"只保留匹配的右表行"应把过滤条件放 WHERE（会变内连接语义，注意是否违背意图）；查"左表没有匹配的行"用 <code>WHERE t2.id IS NULL</code>（LEFT JOIN + 空判断 = 差集）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>LEFT JOIN 右表过滤条件放 ON 和 WHERE 的结果差异（补 NULL vs 丢行）；三个表 LEFT JOIN 的执行顺序（左结合，先左后右）；JOIN 字段类型不一致会怎样（隐式转换，可能索引失效）。</div>
  </div>
</div>

## 七、高可用与扩展

---

# 7.1 数据库读写分离如何实现？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>读写分离 = 主库负责写，从库（只读副本）负责读，主库通过 binlog 把变更复制到从库；应用或代理按 SQL 类型路由（写走主、读走从），分摊读压力。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">实现流程（三层路由）：</b>① <b>应用层</b>：ORM/数据源切分（ShardingSphere、MyCat、Spring 多数据源注解）自动路由；② <b>代理层</b>：数据库中间件（ProxySQL、MyCat）解析 SQL 路由，应用无感；③ <b>驱动层</b>：JDBC 驱动级（ReplicationDriver）。复制链路：主库 binlog → 从库 I/O 线程拉取 → relay log → SQL 线程重放。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">关键细节（写后读一致）：</b>主从延迟是核心问题——刚写入就读的数据要路由到主库、会话粘滞或等待副本追平；<b>事务内的读必须走主库</b>；GTID 便于断点续传；监控延迟秒级告警。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">风险与取舍：</b>延迟导致读到旧数据（可接受最终一致才路由从库）；从库故障要摘除自动切换；复制中断要报警；读写分离不解决写瓶颈（写放大仍需分库分表）；不会自动提升所有查询性能——复杂 Join、报表和大事务仍可能压垮副本，热点数据配合缓存。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>主从延迟怎么缓解（读主/等位点/半同步）；事务内为什么必须读主（隔离性）；主库故障切换怎么防脑裂（选主协议 + 半同步）。</div>
  </div>
</div>

---

# 7.2 分库分表怎么做？分片 key 怎么选？雪花算法时钟回拨怎么办？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯WXG、字节、乐信</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理（什么时候才分）：</b>只有单实例<b>容量、写入吞吐或维护窗口</b>成为硬瓶颈时才分（两千万行通常不用）；分库 = 按业务拆多个库，分表 = 单表拆多张；常见拆分：垂直拆分（按业务模块/字段冷热）→ 水平拆分（按分片键数据分散）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分片 key 选择原则：</b>① 选<b>高频等值查询条件</b>（如 user_id/order_id）——保证同一实体的数据在同一分片，单分片内完成大部分查询；② 分片键要<b>分布均匀</b>（避免热点）；③ 分片方式：<b>哈希取模</b>（<code>user_id % N</code>，均匀但扩容要迁移）、<b>范围分片</b>（按时间/ID 区间，扩容简单但可能热点）、<b>一致性哈希</b>（迁移量小）；④ 关联查询多按同 key 分片，跨分片查询用冗余表或汇总。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分库分表的缺陷与补偿：</b>跨分片事务（尽量收敛单分片，必要时 2PC/TCC/最终一致）、全局唯一 ID（雪花/号段）、跨分片 JOIN（冗余/应用层组装/大宽表）、扩容迁移（双写 + 影子迁移）、分片路由（中间件 ShardingSphere/MyCat）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">雪花算法时钟回拨：</b>雪花 ID = 时间戳 + 机器 ID + 序列号，时钟回拨会导致 ID 重复或倒退。处理：① <b>回拨时间短</b>（< 5ms）：等待时钟追平再继续（自旋）；② <b>回拨中等</b>：拒绝生成并告警，或用<b>备用时钟源</b>；③ <b>大回拨</b>：用<b>历史最大时间戳</b>兜底（ID 不严格递增但唯一），或切换机器 ID；④ 预防：NTP 配置 maxslew 限制时钟跳变。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>取模分片扩容怎么迁（2 倍扩容法：N → 2N 只迁移一半数据）；分片后怎么按非分片键查询（索引表/基因法）；雪花序列号耗尽怎么办（等待下毫秒或扩展位）。</div>
  </div>
</div>

---

# 7.3 垂直拆分和水平拆分怎么选？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">垂直拆分：</b>按业务域或字段冷热拆开，例如把用户、订单、支付分别放到不同库，或把大字段拆到扩展表。它主要解决<b>职责边界、单表宽度、连接与资源隔离</b>问题，但跨库调用和分布式事务会增加。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">水平拆分：</b>同一张逻辑表按分片键拆成多张物理表或多个库，例如 <code>user_id % 64</code>。它主要解决<b>单表容量、索引体积、写入吞吐和并发连接</b>问题，但会带来路由、扩容、跨分片查询和全局 ID 的复杂度。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">选择顺序：</b>先通过索引、SQL、归档、读写分离和缓存解决局部问题；确认单实例容量、写入吞吐或维护窗口成为硬瓶颈后，优先按业务边界做垂直拆分，再按稳定且高频的查询键做水平拆分。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">代价判断：</b>如果核心事务经常同时修改多个业务域，过早垂直拆分会把本地事务变成分布式事务；如果查询大多按用户维度且数据持续增长，水平拆分更合适。最终要用访问模式、数据规模、增长速度和运维能力一起评估。</div>
  </div>
  <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>仅做垂直拆分为什么往往解决不了热点写入问题？如何判断某个查询是否值得冗余字段或建立汇总表？</div>
</div>

---

# 7.4 分片键怎么选才能避免热点？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">基本原则：</b>分片键同时要满足<b>高基数、分布均匀、长期稳定、查询常携带</b>四点。高频等值查询优先选择 <code>user_id</code>、<code>order_id</code> 等业务键，使请求能路由到单个分片；不能只追求均匀而牺牲查询命中。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">热点识别：</b>先按分片统计 QPS、写入量、锁等待、磁盘和缓存命中率，识别“少数 key 占大多数流量”的情况。时间戳、固定租户 ID、递增序列直接做分片键，容易让新数据或大客户集中到一个分片。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">降热点手段：</b>可对天然热点业务键增加<b>哈希扰动或多桶</b>，例如 <code>bucket = hash(order_id) % K</code>；读请求再并行聚合多个桶。对超大租户可单独分片或租户内二次分片，但要同步修改路由、唯一约束和数据迁移策略。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">不能盲目随机：</b>随机分片虽然均匀，却会让按用户查询广播到所有分片；因此应根据主要访问路径在“单分片命中”和“写入均匀”之间取平衡，并为非分片键查询准备索引表、冗余字段或搜索索引。</div>
  </div>
  <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>如果业务天然按商户聚集，热点商户怎么做专项治理？分片键变更时如何迁移历史数据？</div>
</div>

---

# 7.5 跨分片 join、分页、排序、count 怎么处理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">JOIN：</b>优先让关联表使用相同分片键，使关联数据落在同一分片；无法共分片时，可在应用层先查主表再批量 <code>IN</code> 查关联表，或维护冗余字段、宽表和搜索索引。禁止对每条结果逐条跨库查询，避免产生 N+1 次网络往返。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分页和排序：</b>先根据分片键路由，单分片查询优先使用基于索引的游标分页；必须跨分片时，各分片取前 <code>offset + limit</code> 条，再在应用层做归并排序并截取结果。深分页改用“上一页最后一条记录”的 keyset 游标，避免每个分片扫描大量无效行。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">COUNT 和聚合：</b>各分片先局部聚合，再由协调层求和、最大值或合并集合；精确 count 代价高时维护异步汇总表，允许的场景使用近实时或估算值。聚合查询要设置分片超时、部分结果策略和最大扇出，防止一个慢分片拖垮整体。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">一致性与代价：</b>应用层聚合存在快照不一致、结果乱序和部分失败，需要记录查询时间点、分片成功数和降级标识；复杂分析应下沉到 OLAP、搜索引擎或离线数仓，不让 OLTP 分片承担全表扫描。</div>
  </div>
  <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>跨分片查询延迟高时，先改 SQL 还是先改架构？如何处理一个分片失败时的部分结果和重试？</div>
</div>

---

# 7.6 不停机迁移到分库分表怎么做？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答（五步）：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">1. 设计新拓扑：</b>先确定路由规则、分片键、全局 ID、唯一约束和跨分片查询方案，建立影子表或新库，不让线上流量直接写入未验证的目标。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">2. 双写与回填：</b>发布兼容版本，主库成功写入后通过可靠消息、CDC 或事务内 outbox 写入目标；历史数据按主键范围分批回填，控制批量、限速和锁影响，回填任务必须可暂停、重试和幂等。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">3. 校验与追平：</b>做行数、主键集合、关键字段哈希和业务聚合校验，持续比较增量延迟和失败补偿队列。发现差异先定位是漏写、乱序、类型转换还是分片路由错误，不要直接切流。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">4. 灰度切读切写：</b>先按租户或流量比例灰度读目标，保留旧库对照读和一键回退开关；稳定后再切换写入主路径，观察错误率、P95/P99、数据库负载、复制延迟和跨分片广播次数。</div>
    <div style="margin:8px 0 0;"><b style="color:#9B5C00;">5. 收尾与回滚：</b>旧库至少保留一个完整观察周期，确认补偿队列清零、备份可恢复后再下线。回滚要区分“只回滚读流量”和“已经产生新写入”的情况；双写期间所有操作带业务幂等键，避免来回切换造成重复数据。</div>
  </div>
  <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>双写阶段如何避免主从数据长期漂移？CDC 延迟过高时是否允许切流？迁移过程中遇到唯一键冲突怎么处理？</div>
</div>

## 八、选型与工程实践

---

# 8.1 OLTP、OLAP、HTAP 常用数据库有哪些？怎么选型？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">OLTP（事务型）</b>
        <div style="margin:4px 0 0;">MySQL（最主流）、PostgreSQL（功能强）、TiDB（MySQL 协议 + 分布式扩展）、OceanBase（金融）、SQL Server/Oracle（传统企业）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">OLAP（分析型）</b>
        <div style="margin:4px 0 0;">ClickHouse（实时报表/日志）、Doris/StarRocks（实时分析 + 明细查询）、Trino/Presto（联邦查询）、Snowflake/BigQuery（云数仓）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">HTAP（混合）</b>
        <div style="margin:4px 0 0;">TiDB（行列混合）、OceanBase、SingleStore；PostgreSQL + 扩展方案组合。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">选型原则：</b>交易核心链路优先 OLTP（一致性和延迟优先）；经营分析/报表优先 OLAP（扫描聚合性能优先）；既要实时写又要近实时分析才考虑 HTAP（评估成本和复杂度）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>ClickHouse 为什么分析快（列式存储 + 向量化）；TiDB 怎么做到分布式事务（Raft + Percolator 模型）；HTAP 的代价（资源竞争 + 治理复杂）。</div>
  </div>
</div>

---

# 8.2 MySQL 和 PostgreSQL 有什么区别？怎么选？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL</b>
        <div style="margin:4px 0 0;">工程化和生态成熟，互联网 OLTP 默认选型；能力够用，团队上手快。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">PostgreSQL</b>
        <div style="margin:4px 0 0;">功能"全能型"：高级 SQL、窗口分析、JSONB、数组/范围类型、PostGIS、丰富索引（GIN/GiST/BRIN/表达式/部分索引）；两者都支持 MVCC。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">怎么选：</b>高并发交易 + 常规 CRUD + 团队经验在 MySQL → MySQL；复杂报表/复杂 SQL、地理空间、半结构化数据、强约束需求 → PostgreSQL；大厂常见做法：核心交易链路 MySQL，分析/地理/复杂建模模块用 PostgreSQL（按场景拆分）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>PG 的 JSONB 比 MySQL JSON 好在哪（索引 + 操作符）；两边主从复制差异（异步为主 vs 流复制）；迁移成本怎么评估（SQL 兼容性 + 运维体系）。</div>
  </div>
</div>

---

# 8.3 高并发场景下数据库连接池如何优化？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">maxWait 有限等待</b><br>获取连接最多等多久——无限等待会让请求卡死在线程里，最终拖死应用线程池（假死）。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">connectionTimeout / socketTimeout</b><br>建连超时 + 等待响应超时——不配好，网络异常时连接长期僵死占着池子不释放。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">maxActive 控制上限</b><br>连接数不是越大越好——过大增加数据库 CPU、上下文切换和锁竞争，吞吐反而下降；按压测设定。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">一句话：</b>连接池调优不是"多给连接"，而是"<b>限制等待、设置超时、控制上限</b>"，避免应用线程和数据库一起被拖死。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>连接池满了怎么降级（快速失败 + 排队限流）；Druid/HikariCP 的区别（监控 vs 性能）；连接泄漏怎么查（getConnection 未归还，监控 active 数）。</div>
  </div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
