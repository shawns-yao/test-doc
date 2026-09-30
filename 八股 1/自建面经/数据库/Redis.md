# Redis

## 一、高并发过滤

# 1.1 亿级流量场景下，需要和 30 万个黑名单 ID 做匹配，并要求秒级响应，如何高效实现黑名单过滤？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理（两级方案）：</b>黑名单完整数据存储在 Redis <code>SET</code> 中作为<b>精确判断依据</b>，同时在每个应用实例本地维护 <b>Bloom Filter（布隆过滤器）</b> 做预筛。请求到达后先查本地 Bloom：判断“不存在”直接放行；判断“可能存在”再通过 Redis <code>SISMEMBER</code> 精确确认——绝大多数非黑名单请求在应用内存完成判断，避免每个请求都访问 Redis，大幅降低网络 IO 和 Redis QPS。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">请求处理流程：</b></div>
    <div style="position:relative;padding-left:36px;margin:8px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin-bottom:8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">请求 → 查本地 Bloom Filter</div>
      </div>
      <div style="position:relative;margin-bottom:8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#3F8C12;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">不存在 → <b>直接放行</b>（零 Redis 访问）</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">可能存在 → Redis <code>SISMEMBER</code> 精确确认 → 命中才拦截。Bloom 允许假阳性，最多导致一次额外 Redis 查询，<b>不会直接误拦截</b>。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">同步顺序（关键细节）：</b>必须避免数据同步延迟产生<b>“假阴性”</b>（Bloom 判不存在但实际是黑名单）。<b>新增黑名单</b>：先保证 Bloom Filter 能命中，再更新 Redis——中途失败也只是 Bloom 提前认为“可能存在”，最终 Redis 校验仍正确放行；<b>删除黑名单</b>：先从 Redis 删除，普通 Bloom 不立即删除对应元素，允许暂时保留假阳性，后续通过<b>周期性全量重建</b>清理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分布式最终一致：</b><b>MQ/CDC 增量同步 + 版本号校验 + 周期性全量重建</b>。每个变更事件支持幂等消费；实例发现版本号断层则放弃增量，重新加载完整快照并<b>原子替换</b>本地 Bloom Filter。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">性能扩展路径：</b>先通过本地 Bloom、大批量查询和连接池降低 Redis 压力；只有单个 Redis 实例的 CPU、网络吞吐或 QPS 确实达到瓶颈时，再考虑 Redis Cluster 分片。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">监控指标：</b>Bloom 误判率、Bloom 命中率、Redis <code>SISMEMBER</code> QPS / P99 延迟、数据版本差异、同步失败率。</div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>Bloom 只负责快速排除“不可能命中”的请求，最终是否命中黑名单必须由精确存储确认；同步顺序的核心是“新增先 Bloom、删除先 Redis”，保证不出现假阴性。</div>
</div>

---

# 1.2 Bloom Filter 与 Redis 的双写一致性怎么保证？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：沐瞳、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>不要追求 Bloom Filter 和 Redis 的<b>强一致</b>——Bloom 只是预筛层、Redis 才是精确结果。核心底线是<b>不能出现“Redis 已经是黑名单，但 Bloom 还判断不存在”</b>（假阴性 = 放行真黑名单），假阳性（多查一次 Redis）可以容忍。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">写入顺序（实现流程）：</b>① <b>新增</b>：优先让 Bloom 可命中，再写 Redis——中间失败最多产生假阳性、多查一次 Redis，不会错误放行；② <b>删除</b>：先从 Redis 删除，普通 Bloom 不立即删，允许暂时保留假阳性，后续通过<b>定期全量重建</b>清理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">分布式兜底：</b>MQ/CDC 做增量同步 + <b>幂等消费</b> + <b>版本号校验</b> + <b>周期性全量重建</b>；发现版本断层或同步失败时，重新拉取完整黑名单并<b>原子替换</b> Bloom，保证最终一致。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">面试追问：</b>为什么新增先写 Bloom（防假阴性窗口）；删除为什么不从 Bloom 删（普通 Bloom 不支持删除，保留假阳性换简单）；全量重建什么时候触发（版本断层/定时/误判率超标）。</div>
  </div>
</div>

---

# 1.3 布隆过滤器存在误判，怎么降低误判带来的影响？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">原理：</b>位数组 + 多个哈希函数。“不存在”可确定不存在；“可能存在”仍可能误判。它不会把真实存在的元素判为不存在，适合前置排除，不能直接作为最终授权、黑名单或扣款依据。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">降低影响：</b>采用“布隆过滤器 + 精确存储”两级结构——先排除不可能命中的请求，可能命中时再查 Redis Set、Hash、数据库或本地精确集合。根据元素数量、目标误判率和哈希函数数量计算位数组容量，接近容量上限时重建新版本。普通布隆过滤器不支持安全删除，删除场景使用计数布隆过滤器、版本切换或以精确存储为准。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">监控与边界：</b>监控实际误判率、位数组占用、精确查询比例、更新延迟和内存成本。对撤销黑名单、权限校验等高风险场景，即使过滤器判断不存在，也要按业务要求做最终权威检查。</div>
  </div>
</div>

---

# 1.4 黑名单过滤还有哪些优化手段？（关联 1.1）

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">在 1.1 的「Bloom 预筛 + Redis 精确」基础上，进一步分层优化（布隆与精确校验的具体方案见 1.1）：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">入口拦截</b><br>校验 ID 格式、用户、设备和 IP，明显异常请求在网关限流。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">本地缓存</b><br>服务本地用短 TTL 的 Caffeine 等缓存快速判断，进一步减少 Redis 访问。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">分片扩展</b><br>黑名单很大时按 hash 分片到 Redis Cluster，用 pipeline 减少网络往返。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">热 key 处理</b><br>热点黑名单 key 避免集中在单节点，必要时复制到本地只读结构。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">更新一致性：</b>区分新增、撤销和全量替换，用版本号或双版本切换避免读到半套数据；本地缓存、Redis 和过滤器统一版本，撤销时优先失效本地缓存；更新消息必须幂等、可重试，失败进入死信或补偿任务（同步顺序细节见 1.2）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">红线：</b>最终命中结果必须以精确存储为准，不能让布隆过滤器误判直接造成误拦截。监控包括过滤器误判率、Redis 命中率、P99 延迟、更新失败、版本差异和撤销生效时间。</div>
  </div>
</div>

## 二、数据结构与高并发

---

# 2.1 Redis 和 MySQL 的主从复制分别是怎么进行的？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Redis 主从复制</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">首次同步：</b>副本主动连接主节点，主节点生成或复用 RDB 快照，副本加载后再接收复制积压缓冲区中的增量命令。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">持续复制：</b>同步完成后，主节点把后续写命令持续发送给副本，副本按顺序执行。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">断线与切换：</b>积压缓冲区包含断点命令时部分重同步，否则重新全量同步；Sentinel/Cluster 负责故障检测、选主和切换，但切换期间可能短暂丢写或读旧数据。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL 主从复制</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">链路：</b>主库把已提交事务写入 binlog，从库 I/O 线程拉取并写入 relay log，SQL 线程或并行复制线程读取重放。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">断点续传：</b>GTID 标识事务，简化断点续传和主从切换。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;">两者默认多为异步复制，主库提交成功不等于副本已经追平；强一致场景需要半同步、等待副本确认或读主库，并监控复制延迟、复制错误和数据校验结果。</div>
  </div>
</div>

---

# 2.2 Redis 和 MySQL 的区别是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Redis</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">定位：</b>内存优先的键值数据库，单次访问路径短。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">能力：</b>String、Hash、List、Set、Sorted Set、Bitmap、Stream 等数据结构；支持 RDB、AOF 持久化。</div>
        <div style="margin:4px 0 0;"><b style="color:#3A5FBF;">适用：</b>缓存、计数、排行榜、会话、限流、分布式锁和临时状态。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">限制：</b>容量和成本受内存限制，复杂关系查询、Join、约束和强事务能力不如 MySQL。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">定位：</b>关系型数据库，以表、行、列和 SQL 为核心。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">能力：</b>事务、ACID、索引、Join、约束、权限和复杂查询。</div>
        <div style="margin:4px 0 0;"><b style="color:#3A5FBF;">适用：</b>订单、账户、商品等核心事实数据的持久化来源。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">成本：</b>磁盘 I/O、锁、日志和查询优化，单次访问通常比 Redis 更重。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;">实际系统常让 MySQL 保存最终事实，Redis 承担缓存和高并发读写。使用 Redis 时要设计过期、淘汰、持久化、故障恢复和缓存一致性；使用 MySQL 时要关注索引、事务边界、慢查询、连接池和备份，不能简单地用一个替代另一个。</div>
  </div>
</div>

---

# 2.3 Redis 和 MySQL 的数据一致性如何保证？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">读写路径：</b>MySQL 作为主数据，Redis 作为缓存。读请求先查 Redis，未命中再查 MySQL 并回填；写请求先在 MySQL 事务中提交，成功后删除或更新 Redis。删除缓存比直接更新更简单，但要处理删除失败和并发回填旧值的问题。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">竞态风险：</b>线程 A 更新数据库后尚未删缓存，线程 B 读到旧缓存；A 删缓存后，线程 C 因主从延迟读到旧数据库值，又把旧值写回缓存。对策：短 TTL、延迟双删、版本号、分布式锁、消息队列重试、订阅 binlog 和定期对账——通常只能实现最终一致；消息和补偿任务必须幂等，缓存键最好携带数据版本，避免旧事件覆盖新值。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">强一致边界：</b>余额、库存等强一致数据不能只依赖缓存，应使用数据库事务、唯一约束或条件更新，把 Redis 限制为加速层。监控要覆盖缓存命中率、回源量、删除失败、消息积压、版本差异和最终收敛时间。</div>
  </div>
</div>

---

# 2.4 Redis 事务和 MySQL 事务有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Redis 事务</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">命令：</b><code>MULTI</code>、<code>EXEC</code>、<code>DISCARD</code> 和 <code>WATCH</code>。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">特点：</b><code>MULTI</code> 后命令入队，<code>EXEC</code> 按顺序执行且不被其他客户端命令插入；<code>WATCH</code> 实现乐观锁式条件提交。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">局限：</b>没有 MySQL 那样完整的回滚机制，入队语法错误、执行类型错误和业务条件失败需分别处理。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL 事务</b>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">边界：</b><code>BEGIN</code>、<code>COMMIT</code>、<code>ROLLBACK</code>。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">实现：</b>InnoDB 通过 undo log、redo log、锁、MVCC 和约束实现 ACID；修改可整体回滚，按隔离级别控制并发读写，宕机后能通过日志恢复已提交结果。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;">Redis 事务适合短小的多 key 原子操作或条件更新，Lua 脚本可以把判断和修改放到一次执行中；MySQL 事务适合多行、多表和需要持久化的业务。两者分属两个系统，不能自动组成分布式事务，跨库操作仍需消息、补偿、Saga 或状态机设计。</div>
</div>
</div>

---

# 2.5 Redis 有哪些常用数据结构？底层分别如何实现？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:grid;grid-template-columns:110px 1fr;column-gap:12px;row-gap:6px;margin:6px 0 0;">
      <div style="color:#3A5FBF;font-weight:600;">String</div>
      <div>保存字符串、整数或二进制数据，整数可以直接执行自增，适合缓存、计数和锁值。</div>
      <div style="color:#3A5FBF;font-weight:600;">Hash</div>
      <div>保存字段到值的映射，适合用户、商品等对象；小对象会采用更紧凑的编码，数据变大后转为哈希表。</div>
      <div style="color:#3A5FBF;font-weight:600;">List</div>
      <div>支持两端插入和弹出，适合队列和栈；现代 Redis 通常使用 quicklist 组织多个紧凑列表节点。</div>
      <div style="color:#3A5FBF;font-weight:600;">Set</div>
      <div>保存不重复的无序成员，适合去重、标签和集合运算；底层可能是整数集合或哈希表。</div>
      <div style="color:#3A5FBF;font-weight:600;">Sorted Set</div>
      <div>成员带 score，支持排名和范围查询，常见实现是字典加跳表。</div>
      <div style="color:#3A5FBF;font-weight:600;">Bitmap</div>
      <div>基于 String 的位操作，适合签到、活跃标记和布尔状态。</div>
      <div style="color:#3A5FBF;font-weight:600;">HyperLogLog</div>
      <div>使用固定且较小的空间估算基数，适合允许误差的 UV 统计。</div>
      <div style="color:#3A5FBF;font-weight:600;">Stream</div>
      <div>提供消息 ID、消费组和确认机制，适合轻量消息流。</div>
    </div>
    <div style="margin:8px 0 0;">选择结构要结合访问模式、元素规模、是否需要排序或集合运算、持久化和一致性要求。还要关注单 key 过大、热 key、阻塞命令、过期策略和内存碎片，不能只看命令是否方便。</div>
  </div>
</div>

---

# 2.6 Redis 的 `Sorted Set` 底层是怎么实现的？新旧数据结构如何转换？并发读写和删除失败如何处理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">底层结构：</b>同时维护成员到 score 的字典和按 score、成员排序的跳表。字典用于根据成员快速找到旧 score，跳表用于按分值定位、排名和范围遍历；更新成员时先查字典找旧值，再从跳表删旧插新。跳表增删查平均 <code>O(log n)</code>，字典查找平均 <code>O(1)</code>。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">编码转换：</b>根据元素数量、成员长度和编码条件，在更紧凑的 listpack 结构与 skiplist + dict 结构之间选择；数据增长超过阈值时转为适合大集合操作的结构。阈值随版本和配置变化，重点说“由元素数量和大小触发编码转换”，不要死记固定数字。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">并发与删除失败：</b>单条 <code>ZADD</code>/<code>ZREM</code> 在同一实例内由事件循环连续执行，不会被其他命令插入；多命令组成的业务操作仍可能被穿插，需用 Lua、事务或 <code>WATCH</code>。删除失败要区分 key 不存在、类型错误、网络超时、主从切换和持久化故障，客户端用幂等命令、重试和结果校验，不能因超时就断定命令没有执行。</div>
  </div>
</div>

---

# 2.7 Redis 唯一 ID 怎么设计？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">唯一 ID 通常需要全局唯一、趋势递增、可排序、低延迟生成、无中心单点以及适当长度。常见方案：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">数据库自增</b><br>简单，但跨库扩展困难。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">UUID</b><br>冲突概率低，但无序且较长。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Redis <code>INCR</code></b><br>方便、中心化序列，需考虑持久化和故障切换。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">号段模式</b><br>数据库一次分配一段区间，应用本地递增，减少数据库访问。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">雪花算法：</b>ID 划分为时间戳、机器或数据中心标识和序列号，同一毫秒内由序列号区分并发请求。实现时要处理时钟回拨、机器 ID 分配、序列号耗尽和服务重启；机器标识冲突会产生重复 ID。时间戳位数决定可用年限，序列位数决定单毫秒峰值，按真实 QPS 和部署规模计算。</div>
    <div style="margin:8px 0 0;">选型要结合是否需要按创建时间排序、长度限制、跨地域部署、隐私和迁移需求。数据库仍应设置唯一约束，消息和接口还要带业务幂等键，避免客户端重试导致同一个业务被重复创建。</div>
  </div>
</div>

---

# 2.8 Redis 分布式锁怎么实现？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">加锁：</b><code>SET lock_key token NX PX ttl</code>——只有 key 不存在时设置成功，同时写入过期时间；<code>token</code> 必须是随机且唯一的持有者标识。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">释放：</b>不能直接 <code>DEL</code>，要用 Lua 脚本比较 value 与自己的 token，匹配后再删除，避免锁过期后被新线程获得、旧线程误删新锁。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">续期：</b>TTL 要覆盖正常临界区但不能无限延长；任务可能超过 TTL 时由持有者定期续期，续期前确认 token 仍属于自己。客户端长时间暂停、网络分区、主从切换都可能让锁与业务状态短暂不一致，关键业务需幂等、状态校验或数据库条件更新兜底。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">工程注意：</b>处理获取超时、退避重试、异常释放、锁粒度、监控和故障恢复。Redis 锁适合短任务互斥，不天然提供跨库事务语义；涉及库存、余额和支付时，优先使用数据库唯一约束、条件更新或可靠协调服务，并明确故障模型和一致性边界。</div>
  </div>
</div>

---

# 2.9 怎么解决秒杀库存超卖问题？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="position:relative;padding-left:36px;margin:8px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">入口控制：</b>验证码、用户和设备限流、黑名单、重复购买校验以及队列削峰控制请求量。</div>
      </div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">Redis 原子扣减：</b>库存预热到 Redis，用 Lua 脚本一次完成“库存大于零、扣减库存、记录用户是否购买”，避免先读后写超卖。脚本成功只代表获得下单资格，不等于订单和库存已最终完成。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #FAAD14;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#B26E00;">异步落库兜底：</b>消息队列异步创建订单或扣减 MySQL 库存，消费者用订单号/请求号幂等；MySQL 用 <code>UPDATE stock SET stock = stock - 1 WHERE id = ? AND stock &gt; 0</code> 条件更新兜底；库存预扣、支付超时、取消订单和回补用明确状态机管理，避免多扣或多补。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;">还要处理 Redis 故障、缓存预热失败、消息积压、数据库主从延迟和热点 key。监控覆盖入口 QPS、限流拒绝、Redis 脚本耗时、库存差异、订单成功率、队列堆积和补偿失败率。秒杀不是简单加一把分布式锁，锁粒度过大反而会把大量请求串行化。</div>
  </div>
</div>

---

# 2.10 Redis 数据结构如何用于存储 DAU？为什么使用 Bitmap，而不是 MySQL 或 `COUNT(*)`？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">方案：</b>用户 ID 映射为相对紧凑的整数后，把“某用户当天是否活跃”映射到 Bitmap 的一个 bit，例如 <code>SETBIT dau:2026-09-07 user_id 1</code>；统计去重活跃人数用 <code>BITCOUNT</code>，按天保存 key，用 <code>BITOP</code> 计算交集和并集。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Bitmap 方案</b>
        <div style="margin:4px 0 0;">空间按位占用，重复写入天然幂等，单条位操作和批量统计速度快，适合只关心布尔状态的 DAU。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>用户 ID 很稀疏时浪费空位，可考虑 Redis Set、Roaring Bitmap 或离线聚合。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">MySQL 方案</b>
        <div style="margin:4px 0 0;">每个用户一行活跃记录会产生更多行和索引维护成本；直接 <code>COUNT(*)</code> 需要扫描符合条件的记录，不能天然消除同一用户的重复访问。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">需要明确 Bitmap 统计的是去重用户数，不是访问次数；还要处理时区、日期边界、用户 ID 类型、key 过期和故障恢复。Redis 结果如果用于财务、审计或核心运营口径，最好通过访问日志或数仓离线结果定期校验。</div>
  </div>
</div>

---

# 2.11 缓存穿透、缓存击穿和缓存雪崩分别是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">缓存穿透</b>
        <div style="margin:4px 0 0;">查询缓存和数据库都不存在的 key，大量无效请求绕过缓存直接打到数据库。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">手段：</b>参数和权限校验、布隆过滤器预判、不存在结果缓存短 TTL 空值、网关限流和异常来源拦截。空值缓存不能永久保存。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">缓存击穿</b>
        <div style="margin:4px 0 0;">一个热点 key 在失效瞬间被大量请求同时回源。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">手段：</b>热点数据逻辑过期 + 后台异步刷新；互斥锁只允许一个线程重建，其他请求短暂等待、读旧值或降级。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">缓存雪崩</b>
        <div style="margin:4px 0 0;">大量 key 同时过期、缓存集群故障或流量整体切换，导致后端被瞬时冲垮。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">手段：</b>TTL 随机抖动、分批预热、多级缓存、限流熔断、服务降级和 Redis 高可用。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">排查时要分别监控无效 key 比例、热点 key 命中率、回源 QPS、过期分布、Redis 可用性、数据库连接池和队列堆积。三类问题往往同时出现，治理方案要和数据库容量、降级策略及恢复流程一起设计。</div>
  </div>
</div>

---

# 2.12 怎么用互斥锁解决缓存击穿？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="position:relative;padding-left:36px;margin:8px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin:0 0 8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">抢锁 + 二次检查：</b>缓存未命中时尝试获取互斥锁，例如 <code>SET lock:key token NX PX 3000</code>。抢锁成功的线程再次检查缓存，确认仍未命中后查询数据库、写入缓存并释放锁；未抢到的线程短暂退避后重试读取，或返回旧值/降级结果。二次检查避免等待期间重复重建。</div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3A5FBF;">安全释放：</b>用 Lua 脚本比较 token 后再删除，防止锁过期后被新线程获得、旧线程误删新锁；锁要设置 TTL，重建过程有超时、异常释放和有限重试，任务可能超 TTL 时做带持有者校验的续期。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">局限：</b>互斥锁只能缓解同一个 key 的并发回源，不能解决数据库本身慢、Redis 故障或大量 key 同时过期。对极热点 key，逻辑过期加后台刷新通常比让大量请求阻塞更合适；还要配合限流、熔断、随机 TTL、预热和监控，观察重建耗时、等待线程数和回源 QPS。</div>
  </div>
</div>

---

# 2.13 Redis 网络编程了解吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">通信模型：</b>TCP + RESP 协议。客户端把命令编码为 RESP，服务端解析、执行并返回响应；事件驱动模型处理连接建立、可读事件、命令执行、写回和定时任务。单条命令在同一实例内不被其他客户端命令插入，因此 <code>INCR</code>、<code>SETNX</code> 等单命令具有原子性。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">客户端优化：</b>连接池复用 TCP 连接；pipeline 一次发送多条命令减少网络往返，但 pipeline 不等于事务；需要条件判断和多步原子操作时用 <code>MULTI/EXEC</code> 或 Lua。长耗时命令、大 key、阻塞命令和慢客户端会占用事件循环，通过拆分、分页、异步处理和超时控制降低影响。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">排查：</b>区分客户端排队、网络往返、服务端命令执行、持久化和复制延迟；关注连接数、输入输出缓冲区、慢查询、热点 key 和集群重定向，不能只看平均耗时。</div>
  </div>
</div>

---

# 2.14 Redis 或数据库多个节点之间如何处理数据一致性？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">先定目标：</b>明确需要一致读、顺序一致还是最终一致，以及可接受的延迟和丢失窗口。Redis 主从和 MySQL 副本通常是异步复制，主节点提交成功不代表所有副本已追平。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">刚写入必须读新值：</b>固定读主、会话粘滞、等待副本达到指定复制位点，或携带版本号检查；允许延迟的查询才路由到副本。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">最终一致实现：</b>消息队列、binlog CDC、重试、死信、定期对账和版本校验闭环。变更事件带业务主键、版本或单调序列，消费者按版本丢弃旧消息，重复消息幂等处理；缓存更新失败记录补偿任务，不能静默忽略。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">强一致边界：</b>强一致数据收敛到一个权威写入点，使用事务、唯一约束、条件更新或共识协调服务；网络分区和故障切换时明确可用性与一致性的取舍，设计脑裂防护、选主、未完成事务处理、恢复后的数据校验和回滚。不能把多个节点“都写成功”简单当成分布式事务。</div>
  </div>
</div>

---

# 2.15 基于 Redis 协议的数据库了解吗？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">本质：</b>兼容 RESP 和部分 Redis 命令，使已有客户端和业务代码低成本迁移；底层可能使用磁盘、内存分层、LSM Tree 或分布式存储，容量、成本、持久化和故障恢复能力与单机 Redis 不同。兼容协议不代表所有数据结构、脚本、事务、发布订阅、过期和集群语义都完全一致。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">评估清单：</b>做命令兼容性、数据类型、TTL、Lua、事务、Stream、Pipeline、故障切换和备份恢复测试；用真实 key 分布和读写比例测 P95/P99 延迟、吞吐、存储放大、扩容时间和恢复时间。迁移前确认客户端重连、监控、权限、双写、数据校验和回滚方案，不能只验证几个 <code>GET/SET</code>。</div>
  </div>
</div>

---

# 2.16 RocksDB、Redis 和其他 KV 存储有什么区别？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">RocksDB</b>
        <div style="margin:4px 0 0;">嵌入式 KV 引擎，LSM Tree：写入先落 WAL 和 MemTable，再刷成 SSTable。适合本地状态、写密集和需要磁盘容量的场景。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>应用需自己管理进程生命周期、键编码、二级索引、备份、服务化和访问权限；关注 Compaction、写放大、读放大和 Block Cache。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Redis 与其他 KV</b>
        <div style="margin:4px 0 0;">Redis 是独立的内存优先服务，提供丰富数据结构、过期、网络协议、复制和低延迟访问，适合缓存、计数、会话、排行榜和在线状态。</div>
        <div style="margin:4px 0 0;">其他 KV 可能基于 B+ 树、LSM Tree、列族或共识协议，分别在范围查询、写吞吐、强一致、容量、扩容和运维成本上有不同取舍。</div>
      </div>
    </div>
    <div style="margin:10px 0 0;">选择要看读写比例、点查与范围查询、数据规模、延迟目标、事务、复制、故障恢复、成本和团队能力。不能只比较单次 <code>GET</code> 的 QPS，还要验证升级、备份、恢复、节点故障和数据校验流程。</div>
  </div>
</div>

---

# 2.17 如何设计一个结构来存储 QQ 号和在线状态？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Bitmap</b><br>只判断是否在线且 ID 紧凑时：<code>SETBIT online user_id 1</code> / 离线清零 / <code>GETBIT</code> 查询。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Set</b><br>用户 ID 稀疏时，Bitmap 浪费空间，改用 Set 存在线用户。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Hash / 集合</b><br>需要保存设备、登录时间、地域和连接节点时，按用户维度存详细状态。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">状态维护：</b>在线状态不能只靠一次登录写入。客户端定期发送心跳并刷新 TTL，连接断开或超时后由连接服务清理；多端登录时记录设备集合或连接计数，只有最后一个连接退出才变为离线。更新事件携带连接版本或时间戳，避免网络抖动和乱序消息让旧连接覆盖新状态。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">边界：</b>Redis 只保存实时状态，登录历史和审计信息异步写入持久化存储。还要考虑热 key、集群分片、批量心跳、Redis 故障和短暂不可用时的降级，不能简单把“缓存读不到”判定为用户离线。</div>
  </div>
</div>

---

# 2.18 点赞数据采用消息队列、Redis 和 MySQL 双写时，是否可以不落 MySQL？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">判断依据：</b>看点赞关系的业务价值、恢复要求和可接受丢失量。只是短期互动计数、少量丢失可接受且有可靠持久化和异步导出机制时，Redis 承担高频写入、定期汇总到数仓即可；若点赞影响推荐、排行榜、风控、用户权益或审计，就必须保留可恢复的持久化事实。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">常见方案：</b>Redis 保存即时计数和是否点赞，消息队列承接变更事件，消费者幂等写入 MySQL。事件带唯一 ID、用户 ID、对象 ID、操作类型和版本，消费失败进入重试和死信；定期对账比较 Redis、MySQL 和消息日志。MySQL 保留点赞关系，Redis 负责实时读，缓存失效后还能重建。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">只依赖 Redis 的风险：</b>节点故障、AOF/RDB 恢复点、消息丢失和误删会导致计数回退或关系丢失。即使业务允许最终一致，也应明确保留周期、恢复流程、对账指标和最大可接受数据损失。</div>
  </div>
</div>

---

# 2.19 Redis 中的双写一致性有哪些常见实现方式？先写 MySQL 再删 Redis 可能出现什么问题？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度、腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">常见模式：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">Cache Aside</b><br>读未命中查库回填，写时更新库再删缓存。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">先 DB 后删缓存</b><br>生产最常用，MySQL 为事实来源，写成功删 Redis。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">先删缓存后更新</b><br>更新期间读请求可能回填旧值，常配合延迟双删。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">binlog 异步刷新</b><br>订阅 binlog 或变更消息异步更新缓存，需可靠事件、重试和对账。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">“先写 MySQL 再删 Redis”的竞态：</b>两个窗口——线程 A 更新数据库但尚未删缓存时，线程 B 读到旧缓存；A 删缓存后，线程 C 因读取旧数据库副本或事务快照，又把旧值回填到 Redis。短 TTL、延迟双删、版本号、读主库、分布式锁和缓存写入版本比较可降低概率，但不能无条件宣称强一致。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">强一致建议：</b>关键读写走数据库事务或同一协调机制，牺牲部分性能换取确定性。方案必须覆盖删除失败、重复消息、主从延迟、缓存故障、回填旧值和补偿任务自身失败等场景。</div>
  </div>
</div>

---

# 2.20 Redis 集群中的 16384 个槽位是怎么分配和迁移的？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">槽位映射：</b>key 映射到 <code>0</code>~<code>16383</code> 哈希槽，通常计算 <code>CRC16(key) mod 16384</code>；使用 <code>{tag}</code> 时只对大括号内内容计算哈希，使相关 key 落到同一槽位并支持多 key 操作。每个主节点负责一部分槽位，副本复制主节点数据并在故障时参与转移。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">MOVED / ASK：</b>客户端缓存槽位到节点的映射；请求发到错误节点返回 <code>MOVED</code>，客户端更新映射后重试；迁移过程中可能返回 <code>ASK</code>，客户端只把当前请求临时转发到目标节点，不立即永久修改映射。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">迁移流程：</b>扩容时按批次迁移槽位和 key，等待目标节点接收并校验后再切换槽位归属，期间控制带宽、批量大小和业务影响。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">约束与关注：</b>多 key 命令、事务和 Lua 通常要求 key 位于同一槽位，key 设计要合理使用 hash tag。扩容缩容还要关注复制追平、热点槽位、迁移失败重试、槽位覆盖率、MOVED/ASK 次数和故障恢复。</div>
  </div>
</div>

---

# 2.21 Redis 在项目中怎么使用？缓存失效时间如何设置？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">缓存</b><br>用户、配置和热点查询结果，采用 Cache Aside。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">临时状态</b><br>会话、验证码、幂等键和分布式锁，用带 TTL 的 String 或 Hash。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">高并发原子操作</b><br>计数、排行榜、集合去重和在线状态，分别用自增、Sorted Set、Set、Bitmap。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">TTL 策略：</b>不能一刀切。短期验证码和幂等键按业务有效期设置；热点缓存设置较长 TTL 并使用随机抖动，避免大量 key 同时过期；必须及时失效的数据用主动删除、版本号或消息通知。缓存重建要防止击穿，删除失败要有重试和对账，不能把 TTL 当作一致性方案的唯一保障。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">关注：</b>缓存穿透、热 key、大 key、内存淘汰、主从延迟、持久化和故障降级。上线前通过命中率、回源 QPS、P99 延迟、内存使用和过期分布判断 TTL 是否合理，而不是只看缓存是否命中。</div>
  </div>
</div>

---

# 2.22 NoSQL 和 KV 存储有什么区别？各自适合什么场景？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：腾讯</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念：</b>NoSQL 是相对关系数据库的一类非关系型数据库总称，包含 KV、文档、列族和图数据库等模型。KV 把 key 映射到 value，接口简单、点查性能稳定，但通常不擅长复杂条件查询和多实体关系——KV 是 NoSQL 的子类，不能把两者当成完全并列的技术。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">KV</b><br>会话、缓存、配置、计数、设备状态和按唯一 ID 点查。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">文档</b><br>结构变化较多、以聚合文档读取为主的数据。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">列族</b><br>海量写入和按主键或时间范围访问。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">图</b><br>关系遍历场景。</div>
    </div>
    <div style="margin:8px 0 0;">选择时要评估查询模式、事务、一致性、索引、容量、扩容、故障恢复和运维成本。业务模型应先由访问模式驱动，而不是先选数据库再强行适配；需要复杂 Join、约束和多行事务时仍应优先关系数据库，NoSQL 常作为特定访问路径的存储或缓存补充。</div>
  </div>
</div>

---

# 2.23 Redis 的持久化机制有哪些？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">RDB（Redis Database）快照</b>
        <div style="margin:4px 0 0;">命名强调把某一时刻的整个数据库生成二进制快照；满足保存条件后执行后台快照，或手动触发。文件紧凑、恢复速度通常较快，对运行时写入影响相对可控。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>两次快照之间故障可能丢失最近数据；适合备份、快速恢复和低成本传输。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">AOF（Append Only File）日志</b>
        <div style="margin:4px 0 0;">命名强调把变更命令以追加方式写入文件；按同步策略（每秒、每次写入或由操作系统决定）刷盘，重写按当前数据状态生成更短的等价命令序列。恢复时重放命令；数据丢失窗口取决于 <code>appendfsync</code> 策略。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>日志不断增长，需要重写压缩；更关注较小的数据丢失窗口。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;">两者不是绝对的“一个安全、一个不安全”：RDB 也可能因快照周期丢数据，AOF 也会受磁盘故障、刷盘策略和重写过程影响。生产配置要结合数据价值、写入量、恢复时间目标和磁盘容量；持久化不能替代高可用和备份，还要规划文件目录、磁盘空间、重写期间额外内存、恢复演练、主从复制和备份保留周期。缓存型数据可接受丢失，订单或权益数据应以 MySQL 等权威存储为准。</div>
  </div>
</div>

---

# 2.24 Redis 内存满了怎么办？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <span style="display:inline-block;padding:2px 10px;border-radius:999px;background:rgba(58,95,191,.10);border:1px solid rgba(58,95,191,.22);color:#3A5FBF;font-size:12.5px;font-weight:600;">公司：百度</span>
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">先确认原因：</b>是 <code>used_memory</code> 达到 <code>maxmemory</code>，还是 RSS、内存碎片、持久化 fork 或宿主机整体内存不足。检查 <code>INFO memory</code>、<code>MEMORY STATS</code>、大 key、热 key、客户端缓冲区、复制缓冲区、AOF/RDB 重写和淘汰统计，不能只执行 <code>FLUSHDB</code>；线上保留变更前后的监控和采样，避免误删关键数据。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">缓存场景：</b>配置合适的 <code>maxmemory-policy</code>（按 TTL 或访问频率淘汰），给不同业务设置合理 TTL；清理无效 key、压缩 value、拆分大 key、限制单 key 列表和批量删除；可扩容、增加分片或把低频数据迁移到持久化存储，但扩容前评估复制、迁移和热点分布。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">非缓存场景：</b>保存任务、订单状态等不能随意淘汰的数据时，先暂停非关键写入、扩容或迁移，不能直接开启随机淘汰。检查碎片率、fork 期间写时复制额外内存、AOF 重写和备份空间，必要时错峰执行。</div>
    <div style="margin:8px 0 0;">长期治理要建立 key TTL、容量预算、增长趋势、淘汰命中、写入拒绝和恢复演练的监控闭环。</div>
  </div>
</div>

---

# 2.25 Redis 为什么快？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">基于内存</b><br>数据全在内存，无磁盘 I/O（对比 MySQL 需读页）。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">单线程事件循环</b><br>命令串行执行，天然无锁竞争、无上下文切换。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">高效数据结构</b><br>哈希表、跳表、压缩结构（listpack/ziplist）针对场景优化。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#3F8C12;">I/O 多路复用</b><br>epoll 事件驱动，单线程扛海量连接；pipeline 减少 RTT。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>单线程不怕慢命令吗（怕，大 key/阻塞命令会拖住全实例）；为什么不用多线程（瓶颈在内存和网络，不在 CPU；6.0 后多线程只用于网络读写）。</div>
  </div>
</div>

---

# 2.26 Redis 的 HyperLogLog 和 Geo 是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">HyperLogLog（去重计数）</b>
        <div style="margin:4px 0 0;">做<b>基数统计</b>（UV、独立设备数），结果是近似值；内存极省（每个 key 约固定 <b>12KB</b>），适合海量去重计数。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">代价：</b>标准误差约 0.81%，不能做精确计费；命令 <code>PFADD</code>/<code>PFCOUNT</code>/<code>PFMERGE</code>。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Geo（地理位置）</b>
        <div style="margin:4px 0 0;">存经纬度并做"附近的人/店/骑手"查询；底层 = <b>Sorted Set + GeoHash 编码</b>。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">命令：</b><code>GEOADD</code>/<code>GEODIST</code>/<code>GEOSEARCH</code>（新版推荐）；场景：外卖附近商家、打车最近司机。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">一句话：</b>HLL 解决"海量去重计数省内存"，Geo 解决"经纬度存储和附近检索"——前者重统计，后者重空间查询。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>HLL 为什么省内存（概率结构，不存元素本身）；Geo 为什么能范围查询（GeoHash 编码后按 score 排序）；HLL 能精确计数吗（不能，误差 0.81%）。</div>
  </div>
</div>

---

# 2.27 从海量 key 里查出某一固定前缀的 key？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">低频排查/运维：SCAN</b>
        <div style="margin:4px 0 0;"><code>SCAN 0 MATCH prefix:* COUNT 1000</code>——<b>不要用 KEYS</b>（海量 key 会卡死 Redis 主线程）。</div>
        <div style="margin:4px 0 0;"><b style="color:#3F8C12;">特点：</b>非阻塞、渐进遍历；结果不是强一致快照，可能有重复/漏读（业务要容忍）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">高频业务查询：二级索引</b>
        <div style="margin:4px 0 0;">给 key 前缀单独建 Set/ZSet 二级索引——写入业务 key 时同时写索引集合，查询走索引不扫全库。</div>
        <div style="margin:4px 0 0;"><b style="color:#B26E00;">注意：</b>配合 Lua 原子更新、脏索引清理。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>KEYS 为什么危险（O(N) 阻塞单线程）；SCAN 游标是什么（每次返回新游标继续迭代）；重复/漏读怎么处理（业务幂等 + 容忍）。</div>
  </div>
</div>

---

# 2.28 Redis 哨兵机制是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念：</b>Sentinel 解决主从模式 <b>master 宕机后服务不可用</b>的问题——独立进程，监控多个 master-slave 集群，通过<b>多哨兵投票确认故障并自动选主切换</b>，保证主节点故障后服务可恢复。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">选型：</b>高可用用 <b>Sentinel</b>；要横向扩容/分片用 <b>Redis Cluster</b>——Sentinel 管"主挂了自动切"，Cluster 管"数据分散 + 高可用"。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>哨兵怎么判断主节点挂了（主观下线 + 多哨兵客观下线投票）；切换期间会丢写吗（可能，异步复制未同步部分）；客户端怎么感知切换（哨兵通知 + 客户端重连新主）。</div>
  </div>
</div>

---

# 2.29 一致性哈希和虚拟节点是什么？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">一致性哈希：</b>对 <code>2^32</code> 取模，把哈希值空间组成虚拟圆环；节点按 IP/主机名哈希定位置；key 沿环<b>顺时针</b>找到第一个节点作为归属——新增/下线节点只影响相邻区间数据，迁移量显著降低。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">数据倾斜问题：</b>节点很少时分布不均，大量数据集中到某台服务器——为每台服务器计算<b>虚拟节点</b>均匀分布到环上（多一步虚拟节点 → 实体节点映射，数据定位算法不变），相对较少的数据节点也能均匀分布（实际常用 32 个虚拟节点）。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">对比：</b>Redis Cluster 用固定 16384 槽位 + CRC16（见 2.19），一致性哈希常用于代理层分片（如客户端一致性哈希、Codis）；两者都是"key → 节点"的映射，区别在映射规则和迁移策略。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>一致性哈希迁移量为什么小（只影响相邻区间）；虚拟节点解决什么（数据倾斜）；和取模分片的区别（扩容只迁部分 vs 全量重分布）。</div>
  </div>
</div>

---

# 2.30 大 key 的风险与治理？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">本质问题：</b>一次命令处理数据太多，放大 Redis <b>单线程模型</b>的时延风险。大 key 会导致：单次操作阻塞事件循环、网络包过大、主从复制耗时长、集群槽位热点。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">治理四原则：</b></div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">① 拆分 key（核心）</b>：按维度/时间拆小</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">② 分页访问</b>：<code>HSCAN</code>/<code>SSCAN</code>/<code>ZSCAN</code>、<code>ZRANGE</code> 分页，避免一次全拉</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">③ 异步删除</b>：<code>UNLINK</code> 替代 <code>DEL</code>（后台回收）</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">④ 监控发现</b>：<code>--bigkeys</code> 扫描 + 内存分析</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>热 key 和大 key 的区别（访问热点 vs 体积大，可同时存在）；UNLINK 为什么不阻塞（后台线程回收内存）；拆 key 影响业务怎么办（聚合层/多 key 查询补偿）。</div>
  </div>
</div>

---

# 2.31 Redisson 分布式锁的优势？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;">生产推荐直接用 <b>Redisson</b>（比手写 <code>SET NX PX</code> 更完整，底层仍是 Redis 锁语义）：</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">可重入</b><br>同一线程重复加锁不会自己锁死（计数）。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">看门狗续期</b><br>业务没执行完自动延长过期时间，防锁提前失效。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">安全解锁</b><br>内部 Lua 校验锁持有者，避免误删别人的锁。</div>
      <div style="flex:1 1 210px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:var(--primary);">高效等待</b><br>Pub/Sub 通知唤醒，减少无脑轮询。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">API 完整：</b>公平锁、读写锁、联锁、红锁等开箱即用。手写方案的细节（Lua 释放/token）见 2.8。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>看门狗原理（默认 30s，每 10s 续期，业务结束释放）；主从切换时锁会丢吗（会，RedLock 解决但性能差，多数场景可接受）；锁续期失败怎么办（业务幂等兜底）。</div>
  </div>
</div>

---

# 2.32 为什么不把所有查询都塞到 Redis？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">内存成本</b><br>全量数据进内存成本极高（对比磁盘存储）。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">一致性复杂</b><br>全量缓存要维护与数据库的一致性，更新/失效链路变长。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">失效治理难</b><br>全量 key 的过期、淘汰、重建策略复杂，运维成本高。</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:8px 10px;font-size:13px;line-height:1.6;"><b style="color:#B26E00;">冷数据浪费</b><br>低频数据占内存，命中率低，性价比差。</div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">原则：</b>Redis 适合<b>高频热点和低延迟读</b>；冷热分层 + 按访问特征选型（MySQL 存全量事实，Redis 存热点，ES 存检索）更可持续。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>什么数据适合放 Redis（热点/短 TTL/计数类）；冷数据怎么处理（归档/淘汰/降级存储）。</div>
  </div>
</div>

---

# 2.33 Redis Hash 的渐进式扩容？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">扩容触发：</b>hash 内部元素拥挤（碰撞频繁）时扩容——申请<b>两倍大小</b>新数组，把键值对重新分配（rehash）。如果 hash 很大（上百万键值对），一次完整 rehash 耗时很长，对单线程 Redis 压力大。</div>
    <div style="margin:8px 0 0;"><b style="color:#3A5FBF;">渐进式 rehash：</b>同时保留<b>新旧两个 hash 结构</b>，在定时任务和读写指令中<b>逐步迁移</b>旧结构元素到新结构——避免扩容导致的线程卡顿。读写先查新表，查不到再查旧表；迁移完成后释放旧表。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">缩容：</b>Redis 的 hash 还有缩容（比 Java HashMap 更完整）——原理同扩容，新数组比旧数组小一倍。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>渐进式 rehash 期间数据在哪个表（新表优先 + 旧表兜底）；为什么不能一次完成（阻塞单线程事件循环）；和 Java HashMap 扩容区别（Java 一次性扩容 vs Redis 渐进）。</div>
  </div>
</div>

---

# 2.34 Redis 实现限流（zset 滑动窗口）？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3F8C12;">原理（zset 滑动窗口）：</b>每次请求写入一个 zset——<code>value</code> 用 UUID 保持唯一，<code>score</code> 用当前时间戳；判断当前时间窗口内的请求数 = 统计 <code>score</code> 在两个时间戳之间的元素数量（<code>ZCOUNT</code>），超过阈值拒绝；过期窗口用 <code>ZREMRANGEBYSCORE</code> 清理。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">优点：</b>窗口平滑、精度高，适合"最近 N 秒最多允许 M 次请求"的限流规则（区别于固定窗口的临界突刺）。</div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">追问：</b>固定窗口和滑动窗口区别（临界双倍请求 vs 平滑）；zset 限流的缺点（每请求一条记录，内存增长——用 TTL 或定期清理）；更轻量的方案（INCR + 过期 = 固定窗口；Lua 原子执行）。</div>
  </div>
</div>

---

# 2.35 ZooKeeper 分布式锁怎么做？和 Redis 锁怎么选？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">概念原理：</b>ZooKeeper 锁思路和 Redis 不同，常见做法基于<b>临时顺序节点</b>：</div>
    <div style="position:relative;padding-left:36px;margin:8px 0 0;">
      <div style="position:absolute;left:11px;top:10px;bottom:10px;width:2px;background:var(--border);"></div>
      <div style="position:relative;margin-bottom:8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">1</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">客户端在锁目录下创建临时顺序节点</div>
      </div>
      <div style="position:relative;margin-bottom:8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#3F8C12;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">2</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">如果自己是最小节点，说明拿到锁</div>
      </div>
      <div style="position:relative;margin-bottom:8px;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#9BBBF4;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">3</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">如果不是最小节点，就<b>监听排在自己前面的那个节点</b></div>
      </div>
      <div style="position:relative;margin:0;">
        <div style="position:absolute;left:-29px;top:3px;width:20px;height:20px;border-radius:50%;background:#FAAD14;color:#fff;font-size:12px;font-weight:600;display:flex;align-items:center;justify-content:center;">4</div>
        <div style="background:var(--card);border:1px solid var(--border);border-left:3px solid #B26E00;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;">前驱节点删除后，再尝试拿锁。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">优点：</b>客户端挂掉后临时节点自动删除、锁自动释放；监听前驱节点而非锁根节点，减少<b>羊群效应</b>；更容易实现公平锁语义。</div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">Redis 锁</b>
        <div style="margin:4px 0 0;">性能高、实现轻，适合高频短锁；但底层复制一致性要格外注意（主从异步复制下可能双持锁）。</div>
      </div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3A5FBF;border-radius:12px;padding:10px 12px;font-size:13.5px;line-height:1.7;">
        <b style="color:#3A5FBF;">ZooKeeper 锁</b>
        <div style="margin:4px 0 0;">一致性更强、自动释放更自然，但性能和复杂度成本更高。</div>
      </div>
    </div>
    <div style="margin:8px 0 0;"><b style="color:#B26E00;">选型：</b>高并发热点资源竞争时，更好的方向不是「把锁做得更重」，而是先从业务上拆分资源、降低热点。</div>
  </div>
</div>

---

# 2.36 分布式锁如何抗高并发？

<div style="--primary:#3A5FBF;--card:var(--background-primary);--border:var(--background-modifier-border);--text:var(--text-normal);--muted:var(--text-muted);max-width:780px;font:14px/1.65 Roboto,'PingFang SC','Segoe UI',Arial,sans-serif;color:var(--text);margin:12px 0;box-sizing:border-box;">
  <div style="margin:8px 0 0;background:var(--card);border:1px solid var(--border);border-left:3px solid var(--primary);border-radius:12px;padding:12px 14px;">
    <b>回答：</b>
    <div style="margin:6px 0 0;"><b style="color:#3A5FBF;">问题：</b>热点资源每秒上万请求抢同一把锁，单把锁会成为瓶颈。</div>
    <div style="margin:8px 0 0;"><b style="color:#3F8C12;">优化手段：</b></div>
    <div style="display:flex;flex-wrap:wrap;gap:8px;margin:8px 0 0;">
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">① 分段加锁</b>——把大资源拆成多个小分段，每分段独立加锁</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">② 合并扣减</b>——一个分段不够时，再锁别的分段做组合扣减</div>
      <div style="flex:1 1 300px;min-width:0;background:var(--card);border:1px solid var(--border);border-left:3px solid #3F8C12;border-radius:12px;padding:9px 12px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">③ 无锁化演进</b>——极端高并发直接在 Redis/Tair 这类 KV 里原子扣减，再通过 MQ 异步回写关系型数据库，而不是让所有请求先抢一把重锁</div>
    </div>
  </div>
  <div style="margin:10px 0 0;padding:8px 12px;border-left:3px solid #3F8C12;background:rgba(82,196,26,.08);border-radius:8px;font-size:13.5px;line-height:1.7;"><b style="color:#3F8C12;">口述重点：</b>分布式锁是兜底手段，不是高并发系统的首选主路径；能拆热点、能异步、能原子扣减，就尽量不要把吞吐压在一把锁上。</div>
</div>

---

> 返回导航：[[00-总导航|00-总导航]]
