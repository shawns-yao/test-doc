---
aliases:
- "equals 和 hashCode 为什么必须一起重写"
---

# 03 equals 和 hashCode 为什么必须一起重写

## 01 核心回答

equals 定义逻辑相等，hashCode 帮散列集合缩小候选范围。若两个对象 equals 为 true，它们必须返回相同 hashCode；反向不成立，同一个 hash 可以对应不同对象。

---

## 02 为什么只改 equals 会出错

HashMap 先用 hash 定位桶并筛选候选，再进行 equals 判断。两个业务上相等的键若 hash 不同，就可能落入不同位置，导致查不到或出现逻辑重复。把所有 hash 都返回常量虽然可满足契约，却会造成严重冲突，不是好实现。

equals 应自反、对称、传递、一致，并对 null 返回 false。涉及继承时要警惕父类/子类互认规则不对称；值对象宜有明确的相等边界。数组默认 equals 继承引用身份比较，按元素内容比较应选择 Arrays 的相应方法。

---

## 03 可变键与业务选择

参与相等判断的字段也应决定 hash，并在对象作为散列键期间保持稳定。例如键插入后修改身份字段，后续查找使用新 hash，旧桶里却仍保存原节点。可以设计不可变键，或先删除再以新状态插入。

== 判断两个引用是否同一对象，Object 默认 equals 也如此；重写后 equals 才可能表达值相等。Integer 缓存、String 驻留等都不改变这个原则。

---

## 04 口述与参考

“相等对象必须同 hash，冲突再由 equals 区分；契约错误会破坏查找，分布太差会伤害性能，可变键会破坏入表后的定位。”

- [Object.equals/hashCode 契约](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)
- [[八股/01-Java/02-Java集合/01-HashMap 底层数据结构是什么？如何扩容|HashMap 如何使用 hash 与 equals]]

- [[八股/01-Java/01-Java基础/02-String 为什么不可变？字符串常量池有什么作用|String 的内容相等与驻留身份]]

---

## 05 所属专题

- [[八股/01-Java/01-Java基础/00-Java基础导航|Java基础导航]]
