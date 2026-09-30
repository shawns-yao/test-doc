---
aliases:
- "MyBatis 一级/二级缓存？`#{}` 和 `${}` 的区别？"
- "Spring框架 2.2 MyBatis 一级/二级缓存？`#{}` 和 `${}` 的区别？"
---

# 01 MyBatis 一级缓存和二级缓存有什么区别

## 01 核心回答


**一级缓存（默认开启）**

**SqlSession 级别**：同一 SqlSession 内命中相同缓存键的查询可返回缓存，键还涉及参数、分页和环境等；更新、提交、回滚或关闭等操作会清理本地缓存，localCacheScope 还会影响范围。

**注意：**Spring 集成由 SqlSessionTemplate 管理会话；事务中通常复用绑定的 SqlSession，事务外调用边界更短，不能笼统说一级缓存无效。

**二级缓存（需配置开启）**

**namespace（Mapper）级别**：跨 SqlSession 共享；多表操作要小心脏数据（另一 Mapper 更新后本缓存不失效）——**是否开启需评估**数据变更路径、命中率与一致性需求。

**面试追问：**一级缓存什么时候失效（提交/更新/关闭 SqlSession）；为什么二级缓存可能读到脏数据（其他 namespace 更新不触发本缓存失效）；SQL 参数安全另见 [[八股/02-Spring框架/04-MyBatis/02-MyBatis 的参数绑定与字符串替换有什么区别|参数绑定与字符串替换]]。

---

> 返回导航：[[八股/00-总导航|00-总导航]]

## 02 二级缓存为什么跨会话而不跨所有数据修改

二级缓存通常按 namespace 配置，事务性缓存会在合适的会话提交边界发布数据；同 namespace 的写操作通常使相关缓存失效。其他 namespace、其他应用或手写 JDBC 直接更新同一张表时，MyBatis 不会自动理解全部依赖，所以“同一库里更新了就全局同步失效”不成立。

一级缓存可能返回同一个对象引用，应用修改查询结果可能影响同会话后续读取；避免随意修改共享缓存结果，并理解本地缓存作用域。二级缓存不是数据库隔离级别或分布式一致性的替代。

## 03 面试取舍与参考

先说作用域和生命周期，再说失效边界。读多且更新路径受控时再评估二级缓存；复杂跨服务写入场景应明确缓存失效方案，不能只加 cache 注解就承诺一致。

- [MyBatis 本地缓存与 SqlSession](https://mybatis.org/mybatis-3/java-api.html#Local_Cache)
- [MyBatis namespace 缓存](https://mybatis.org/mybatis-3/sqlmap-xml.html#cache)
- [MyBatis-Spring SqlSessionTemplate 与事务会话](https://mybatis.org/spring/sqlsession.html)
- [[八股/02-Spring框架/04-MyBatis/02-MyBatis 的参数绑定与字符串替换有什么区别|缓存之外的 SQL 参数安全]]

## 04 所属专题

- [[八股/02-Spring框架/04-MyBatis/00-MyBatis导航|MyBatis导航]]
