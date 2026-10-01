# 29 Dijkstra 怎样求非负权最短路并还原路径

## 01 核心 Java 代码

前提：邻接表合法、边权为非负 int；返回源点到终点的路径，不可达返回空列表。

```java
import java.util.*;

// graph[u] 中的边为 {目标点, 非负整数权重}。
static List<Integer> shortestPath(List<int[]>[] graph, int s, int t) {
    int n = graph.length;
    if (s < 0 || s >= n || t < 0 || t >= n) {
        throw new IllegalArgumentException("invalid vertex");
    }
    long[] dist = new long[n];
    int[] parent = new int[n];
    Arrays.fill(dist, Long.MAX_VALUE);
    Arrays.fill(parent, -1);
    PriorityQueue<long[]> pq = new PriorityQueue<>(
        Comparator.comparingLong(a -> a[0]));
    dist[s] = 0;
    pq.offer(new long[]{0, s});
    while (!pq.isEmpty()) {
        long[] cur = pq.poll();
        long d = cur[0];
        int u = (int) cur[1];
        if (d != dist[u]) continue;
        if (u == t) break;
        for (int[] edge : graph[u]) {
            int v = edge[0], w = edge[1];
            if (w < 0) throw new IllegalArgumentException("negative weight");
            long nd = d + w;
            if (nd < dist[v]) {
                dist[v] = nd;
                parent[v] = u;
                pq.offer(new long[]{nd, v});
            }
        }
    }
    if (dist[t] == Long.MAX_VALUE) return Collections.emptyList();
    List<Integer> path = new ArrayList<>();
    for (int v = t; v != -1; v = parent[v]) path.add(v);
    Collections.reverse(path);
    return path;
}
```

---

## 02 复杂度

时间：重复入堆实现为 O(V + E log(E + 1))，还原路径 O(V)。

空间：辅助距离、前驱及结果 O(V)，堆最坏 O(E)，合计 O(V + E)，不含输入邻接表；不是 decrease-key 堆的 O(V) 辅助空间。

---

## 03 相关问题

- [[八股/12-算法/01-算法题/23-岛屿数量怎么做|岛屿数量怎么做]]
- [[八股/12-算法/01-算法题/28-二叉树最近公共祖先（LCA）怎么做|二叉树最近公共祖先（LCA）怎么做]]
- [[八股/12-算法/01-算法题/50-单词接龙怎样按层寻找最短转换|单词接龙怎样按层寻找最短转换]]

---

## 04 来源与改写说明

题目线索：goehou/agent_java_offer 的 Repository contributors，[项目场景题](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/05_项目表达/04_搜索推荐平台/01_核心问答.md)，固定版本 `298656dc4d0fb5f7db107fc6463f11230b3a49f7`。原题线索采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)；本页重新组织为独立问题，并补写 Java 核心方法和复杂度边界，不保留公司押题、命中率或个人履历式表述。许可范围见 [[八股/96-外部资料来源与许可说明|外部资料来源与许可说明]]。

---

## 05 所属专题

- [[八股/12-算法/01-算法题/00-算法题导航|算法题导航]]
