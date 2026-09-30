---
aliases:
- "二叉树最近公共祖先 LCA 怎么做"
---

# 28 二叉树最近公共祖先 LCA 怎么做

## 01 核心代码

一般二叉树，按节点身份比较；两节点存在性检查版不存在则返回 null，也支持 p==q。

```java
import java.util.*;

static class TreeNode {
    int val; TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}

// 简洁版前提：p、q 均在树中。
static TreeNode lca(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lca(root.left, p, q), right = lca(root.right, p, q);
    if (left != null && right != null) return root;
    return left != null ? left : right;
}
static boolean contains(TreeNode root, TreeNode target) {
    return root != null && (root == target || contains(root.left, target) || contains(root.right, target));
}
static TreeNode lcaIfPresent(TreeNode root, TreeNode p, TreeNode q) {
    if (p == null || q == null || !contains(root, p) || !contains(root, q)) return null;
    return lca(root, p, q);
}
// BST 直接变体：键唯一，p/q 已确认存在。
static TreeNode lcaBst(TreeNode root, TreeNode p, TreeNode q) {
    int low = Math.min(p.val, q.val), high = Math.max(p.val, q.val);
    while (root != null) {
        if (root.val < low) root = root.right;
        else if (root.val > high) root = root.left;
        else return root;
    }
    return null;
}
static class ParentNode {
    ParentNode parent;
}
// 父指针无环，可不在同一棵树；无公共祖先返回 null。
static ParentNode lcaWithParents(ParentNode p, ParentNode q) {
    ParentNode a = p, b = q;
    while (a != b) {
        a = a == null ? q : a.parent;
        b = b == null ? p : b.parent;
    }
    return a;
}
```

## 02 时间和空间复杂度

一般树及存在性检查版：时间 O(n)、递归空间 O(h)；BST 版时间 O(h)、额外空间 O(1)；父指针版时间 O(hp+hq)、额外空间 O(1)。

## 03 参考与关联

- [[八股/12-算法/01-算法题/21-二叉树子结构怎么做|二叉树子结构怎么做]]
- [LeetCode 236：自身可作祖先且保证两个节点存在](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)
- [[八股/12-算法/01-算法题/10-如何在二叉搜索树中求第 k 大的数|如何在二叉搜索树中求第 k 大的数]]：搜索树有序性与一般树祖先关系

## 04 所属专题

- [[八股/12-算法/01-算法题/00-算法题导航|算法题导航]]

## 05 相关问题与延伸

- [[八股/12-算法/02-题型清单/04-04_二叉树|04_二叉树]]
- [[八股/12-算法/01-算法题/29-Dijkstra 怎样求非负权最短路并还原路径|Dijkstra 怎样求非负权最短路并还原路径]]
