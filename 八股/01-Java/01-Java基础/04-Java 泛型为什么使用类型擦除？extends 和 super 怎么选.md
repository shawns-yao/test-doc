---
aliases:
- "Java 泛型为什么使用类型擦除？extends 和 super 怎么选"
---

# 04 Java 泛型为什么使用类型擦除？extends 和 super 怎么选

## 01 核心回答

Java 泛型主要把类型检查前移到编译期。编译器将类型参数擦除为上界（未声明时为 Object），必要时插入类型转换和桥接方法，使泛型与旧类库兼容。并非为 `List<String>` 和 `List<Integer>` 各生成一套独立运行时类。

## 02 擦除带来哪些限制

不能直接 new T、new T[]，不能用 instanceof 检查 `List<String>` 的实际元素类型；同名方法若只靠擦除后相同的参数区分，也会签名冲突。反射仍可能读取类/方法声明中的泛型签名元数据，所以“运行时全部泛型信息都消失”同样不准确。

原始类型绕过检查可能造成堆污染，问题延迟到读取时表现为 ClassCastException；不要把 unchecked 警告直接当作可以忽略。编译期安全不是运行时自动检查集合中所有元素。

## 03 为什么 `List<Integer>` 不是 `List<Number>`

若这种赋值成立，接收 `List<Number>` 的代码就能把 Double 插进原来的整数列表，破坏类型安全，因此泛型默认不协变。

只从容器读取 T，常用 ? extends T：可以按 T 读取，但不能任意添加 T，因为实际可能是更窄子类。只向容器写入 T，常用 ? super T：可安全添加 T 及其子类，读取时只保证 Object。PECS 是常用记法，读写都需要确定类型时通常直接用 T；extends 容器并非绝对不可修改，例如仍可能 clear 或 remove。

## 04 参考与关联

- [JLS 类型擦除](https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html#jls-4.6)
- [Oracle 泛型擦除与桥接方法](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html)
- [[八股/01-Java/02-Java集合/03-ArrayList 和 LinkedList 区别？往 ArrayList 中间插入的时间复杂度？怎么优化|集合类型与实现选择]]

## 05 所属专题

- [[八股/01-Java/01-Java基础/00-Java基础导航|Java基础导航]]
