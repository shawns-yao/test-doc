---
aliases:
- "如何求三数之和为 0 的不重复数组？（LeetCode 15，ACM 模式）"
- "手撕代码 1.17 如何求三数之和为 0 的不重复数组？（LeetCode 15，ACM 模式）"
---

# 17 如何求三数之和为 0 的不重复数组？（LeetCode 15，ACM 模式）

## 01 核心代码

返回按数值去重的三元组，允许排序修改输入；ACM 仅补题面指定的输入输出，本页保留核心函数。

```java
import java.util.*;

static List<List<Integer>> threeSum(int[] a) {
    Arrays.sort(a);
    List<List<Integer>> result = new ArrayList<>();
    for (int i = 0; i + 2 < a.length && a[i] <= 0; i++) {
        if (i > 0 && a[i] == a[i - 1]) continue;
        int left = i + 1, right = a.length - 1;
        while (left < right) {
            long sum = (long)a[i] + a[left] + a[right];
            if (sum < 0) left++;
            else if (sum > 0) right--;
            else {
                result.add(List.of(a[i], a[left], a[right]));
                int x = a[left], y = a[right];
                while (left < right && a[left] == x) left++;
                while (left < right && a[right] == y) right--;
            }
        }
    }
    return result;
}
```

---

## 02 时间和空间复杂度

时间 O(n²)；排序工作空间保守 O(log n)，另有 O(r) 输出（三元组数 r）。

---

## 03 参考与关联

- [LeetCode 15：不同下标及三元组去重要求](https://leetcode.com/problems/3sum/)
- [[八股/12-算法/02-题型清单/01-01_数组与双指针|01_数组与双指针]]：反向关联：此题引用了本题的机制或边界

---

## 04 所属专题

- [[八股/12-算法/01-算法题/00-算法题导航|算法题导航]]

---

## 05 相关问题与延伸

- [[八股/12-算法/01-算法题/31-两数之和怎样返回不同位置的下标|两数之和怎样返回不同位置的下标]]
