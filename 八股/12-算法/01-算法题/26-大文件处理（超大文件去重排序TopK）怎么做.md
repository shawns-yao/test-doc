---
aliases:
- "大文件处理（超大文件去重/排序/TopK）怎么做？"
- "手撕代码 1.26 大文件处理（超大文件去重/排序/TopK）怎么做？"
---

# 26 大文件处理（超大文件去重/排序/TopK）怎么做？

## 01 核心代码

记录为非 null 字符串，排序按字典序；每块最多 M 条且总字节受预算限制。外存调度层同步落盘、关闭文件，每轮只打开 F 个有序段，按需多轮归并；以下只给块处理与流式核心。

```java
import java.util.*;

static void makeRuns(Iterator<String> input, int m, java.util.function.Consumer<List<String>> writeRun) {
    if (m <= 0) throw new IllegalArgumentException();
    while (input.hasNext()) {
        List<String> chunk = new ArrayList<>();
        while (input.hasNext() && chunk.size() < m) chunk.add(input.next());
        chunk.sort(String::compareTo);
        writeRun.accept(chunk); // 同步写成一个有序临时段，不在内存保留所有段
    }
}
record Head(String value, int run) {}
static Iterator<String> mergeRuns(List<Iterator<String>> runs) {
    PriorityQueue<Head> heap = new PriorityQueue<>(Comparator.comparing(Head::value));
    for (int i = 0; i < runs.size(); i++)
        if (runs.get(i).hasNext()) heap.offer(new Head(runs.get(i).next(), i));
    return new Iterator<>() {
        public boolean hasNext() { return !heap.isEmpty(); }
        public String next() {
            if (heap.isEmpty()) throw new NoSuchElementException();
            Head h = heap.remove();
            Iterator<String> run = runs.get(h.run());
            if (run.hasNext()) heap.offer(new Head(run.next(), h.run()));
            return h.value();
        }
    };
}
static void distinctSorted(Iterator<String> sorted, java.util.function.Consumer<String> write) {
    String previous = null;
    while (sorted.hasNext()) {
        String x = sorted.next();
        if (!x.equals(previous)) { write.accept(x); previous = x; }
    }
}
static List<Integer> largestValues(Iterator<Integer> input, int k) {
    if (k < 0) throw new IllegalArgumentException();
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    while (input.hasNext()) {
        heap.offer(input.next());
        if (heap.size() > k) heap.poll();
    }
    return new ArrayList<>(heap); // 值最大的 K 条记录，允许重复，结果顺序不限
}
record Frequency(String key, long count) {}
static List<Frequency> topFrequencySorted(Iterator<String> sorted, int k) {
    if (k < 0) throw new IllegalArgumentException();
    Comparator<Frequency> worstFirst = Comparator.comparingLong(Frequency::count)
            .thenComparing(Frequency::key, Comparator.reverseOrder());
    PriorityQueue<Frequency> heap = new PriorityQueue<>(worstFirst);
    String key = null; long count = 0;
    while (sorted.hasNext()) {
        String x = sorted.next();
        if (!x.equals(key)) {
            if (key != null) offer(heap, new Frequency(key, count), k);
            key = x; count = 0;
        }
        count = Math.incrementExact(count);
    }
    if (key != null) offer(heap, new Frequency(key, count), k);
    List<Frequency> result = new ArrayList<>();
    while (!heap.isEmpty()) result.add(heap.poll());
    Collections.reverse(result); // 频次降序，同频 key 升序
    return result;
}
static void offer(PriorityQueue<Frequency> heap, Frequency f, int k) {
    heap.offer(f); if (heap.size() > k) heap.poll();
}
static void intersectSorted(Iterator<String> a, Iterator<String> b,
                            java.util.function.Consumer<String> write) {
    String x = a.hasNext() ? a.next() : null, y = b.hasNext() ? b.next() : null;
    String last = null;
    while (x != null && y != null) {
        int c = x.compareTo(y);
        if (c == 0) {
            if (!x.equals(last)) { write.accept(x); last = x; }
            x = a.hasNext() ? a.next() : null; y = b.hasNext() ? b.next() : null;
        } else if (c < 0) x = a.hasNext() ? a.next() : null;
        else y = b.hasNext() ? b.next() : null;
    }
}
```

---

## 02 时间和空间复杂度

令 N 为记录数、R=ceil(N/M)、F≥2 为归并路数，假设键比较/计数为 O(1)。生成有序段 O(N log(M+1)) 时间、O(M) 内存；每轮归并 O(N log F) 时间、O(F) 工作内存，共 ceil(log_F R) 轮，含初始读写的 I/O 为 O(N(1+ceil(log_F R))) 条记录。已排序流去重 O(N) 时间/O(1) 工作空间；值 TopK O(N log(K+1)) 时间/O(K) 空间；频次 TopK 在全局排序之后为 O(N+u log(K+1)+K log(K+1)) 时间/O(K) 空间。两有序流交集 O(N1+N2) 时间/O(1) 工作空间，以上均不含输入/输出及外存文件。变长键还需乘实际比较成本；Bloom 阳性不能直接丢弃，按文件偏移分块的局部频次 TopK 不能直接当全局答案。

---

## 03 参考与关联

- [CMU 数据库课程：外部归并排序与 B-1 路归并](https://www.cs.cmu.edu/~christos/courses/dbms.S13/slides/13Sorting.pdf)
- [Redis 官方文档：布隆过滤器的误判边界](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/)
- [[八股/12-算法/01-算法题/02-前 K 个高频元素，如何改成求第 2 个高频元素|前 K 个高频元素，如何改成求第 2 个高频元素]]：内存受限时的TopK与外存分治

---

## 04 所属专题

- [[八股/12-算法/01-算法题/00-算法题导航|算法题导航]]

---

## 05 相关问题与延伸

- [[八股/09-系统设计/06-存储与业务系统/16-如何统计最近一小时的 TopK 高频 IP|如何统计最近一小时的 TopK 高频 IP]]
- [[八股/12-算法/01-算法题/60-数组第K大怎样维护容量K的小顶堆|数组第K大怎样维护容量K的小顶堆]]
