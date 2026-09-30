---
aliases:
- "Linux 中 `grep` 和 `awk` 怎么使用？"
- "工具与工程 2.1 Linux 中 `grep` 和 `awk` 怎么使用？"
---

# 01 Linux 中 `grep` 和 `awk` 怎么使用？

## 01 核心回答


**概念原理：**`grep` 负责**按行过滤**（找匹配的行），`awk` 负责**按列处理**（对每行切分字段做计算/格式化）——grep 是筛子，awk 是加工机，常组合使用。常用命令：

```bash
# ===== grep 常用 =====
grep "error" app.log          # 基本匹配
grep -i error app.log         # 忽略大小写
grep -v error app.log         # 反选（不含 error 的行）
grep -n error app.log         # 显示行号
grep -r error ./src           # 递归目录
grep -E "err|warn" app.log    # 扩展正则（多条件）
grep -c error app.log         # 计数
grep -A 3 -B 2 error app.log  # 匹配后 3 行 / 前 2 行

# ===== awk 常用 =====
awk '{print $1, $3}' file            # 取第 1、3 列（默认空格分隔）
awk -F: '{print $1}' /etc/passwd     # 指定分隔符
awk '{sum += $1} END {print sum}'    # 列求和
awk '$3 > 100' file                  # 条件过滤
awk 'NR>1 && $2=="ERROR"' file       # 跳过表头 + 过滤
awk '{print $NF}' file               # $NF = 最后一列

# ===== 组合实战：统计日志中错误码出现次数 =====
grep "ERROR" app.log | awk '{print $NF}' | sort | uniq -c | sort -rn
# grep 过滤 → awk 取列 → 排序计数 → 降序
```

**关键细节：**awk 的 `$0` 是整行；管道是核心思维（一条命令只做一件事）；处理大文件先 `head` 看格式再写完整命令；`sort | uniq -c` 是统计标配。

**面试追问：**grep 和 egrep 的关系（egrep 即 `grep -E`）；awk 和 cut 的区别（awk 能做计算和条件，cut 只能切）；怎么统计每个 IP 的请求数（awk 取 IP 列 + `sort | uniq -c`）。

---

## 02 理解补充与边界校订

grep -c 数的是匹配行，不是每个匹配片段。awk 默认按空白字段处理，含引号逗号或多行的 CSV 不适合直接以逗号 split；JSON 日志也应使用结构化解析器。统计先明确时间范围、重复日志与缺失字段，否则命令跑通也可能算错。大文件 sort 可能外排到磁盘，先检查空间预算。

## 03 依据与延伸阅读

- [GNU grep 计数说明](https://www.gnu.org/software/grep/manual/html_node/General-Output-Control.html)
- [GNU awk 字段](https://www.gnu.org/software/gawk/manual/html_node/Fields.html)

## 04 相关问题

- [[八股/10-工程实践/05-Linux排障/02-Linux CPU 过载时如何排查和优化|Linux CPU 过载时如何排查和优化]]：日志筛查与性能现场分析

## 05 所属专题

- [[八股/10-工程实践/05-Linux排障/00-Linux排障导航|Linux排障导航]]
