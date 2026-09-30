---
aliases:
- "String 为什么不可变？字符串常量池有什么作用"
---

# 02 String 为什么不可变？字符串常量池有什么作用

## 01 核心回答

String 实例创建后其字符序列不能通过公开 API 修改。变量可以重新指向另一个 String，但这不是修改旧实例。不变性让字符串能够安全共享、稳定作为 Map 键，并便于缓存 hash 与复用字面量。

## 02 不可变不是只靠 final

String 类不可被继承，内部存储不向调用者暴露可写引用，构造/操作遵守防御性设计。final 字段只禁止字段重新赋值，若它指向可变数组且把数组直接泄露，类仍可能可变。因此“String 因为类是 final 所以不可变”不完整。

旧版用 char[] 的记忆不能套在所有 JDK 上；现代 HotSpot 常用紧凑字符串的 byte[] 加编码标记作为实现细节。API 仍以 UTF-16 code unit 定义索引，length 不保证等于用户感知字符数，某些字符需要代理项对。

## 03 常量池与比较

字符串字面量和编译期常量表达式可共享驻留实例，intern 返回对应规范化字符串引用；运行时构造的相同文本不保证引用相同。== 比较引用身份，equals 比较字符序列内容。不要用偶然驻留或缓存行为代替 equals。

循环反复拼接可能产生多次中间结果，明确的大量增量拼接可使用 StringBuilder；单个简单表达式的 + 如何降低到字节码由编译器/JDK 决定，不能断言所有 + 永远先生成 StringBuilder。StringBuffer 方法同步也不让多步组合自动原子。

## 04 参考与关联

- [String API：不可变、索引、intern](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)
- [[八股/01-Java/01-Java基础/03-equals 和 hashCode 为什么必须一起重写|稳定键与散列契约]]
- [[八股/01-Java/01-Java基础/01-Java 是值传递还是引用传递|对象状态与引用变量的区别]]

## 05 所属专题

- [[八股/01-Java/01-Java基础/00-Java基础导航|Java基础导航]]
