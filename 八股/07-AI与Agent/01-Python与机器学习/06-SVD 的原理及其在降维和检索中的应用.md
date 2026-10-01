# 06 SVD 的原理及其在降维和检索中的应用

## 01 核心回答

对实数 m×n 矩阵 A，奇异值分解写作 A = UΣVᵀ：U、V 的列给出正交方向，Σ 的对角元素是非负奇异值，通常按从大到小排列。保留最大的 k 个奇异值及对应向量，得到 Aₖ = UₖΣₖVₖᵀ，用较少维度近似原矩阵。

SVD 不要求 A 是方阵。截断后的信息损失取决于丢弃的奇异值及下游任务，不能把 k 当固定“最佳维度”。奇异向量的符号不唯一，不应把不同实现的单列符号差异直接判断为错误。

---

## 02 与 PCA 和文本检索的关系

PCA 对中心化数据寻找主要方差方向，可以通过 SVD 求解。TruncatedSVD 通常不中心化，因此适合稀疏词频或 TF-IDF 矩阵，避免中心化破坏稀疏性；作用在词项表示上通常称为潜在语义分析（LSA）。

若行是文档、列是词项，文档低维表示为 UₖΣₖ；新 query 使用相同词表与预处理，再乘 Vₖ 投影到兼容空间。训练只用训练语料，变更词表或投影矩阵后需同步查询端和文档端，不能混用版本。

---

## 03 推荐场景的边界

用户—物品矩阵可用低秩潜因子解释偏好，但真实评分矩阵大量缺失，缺失不代表评分为零。推荐系统常在已观测条目上优化带正则的矩阵分解目标，不能把这种模型与对填零矩阵直接做标准 SVD 混为一谈。

降维后仍要验证召回、排序、长尾覆盖和资源成本；重构误差变小不一定带来业务收益。新用户或新物品没有足够行为时，纯协同潜因子也不会自动解决冷启动。

---

## 04 直接追问与口述

- k 如何选？结合奇异值谱、验证集质量、索引内存和时延，而不是只追求保留率
- SVD 与特征值分解的关系？AᵀA 的特征值对应奇异值平方；数值实现无需总是显式构造 AᵀA，后者还可能恶化条件数
- 口述：“SVD 将矩阵拆成方向与强度，截断保留低秩结构。检索重点是共享投影空间，推荐重点是正确处理缺失观测。”

---

## 05 相关问题与导航

- [[八股/07-AI与Agent/01-Python与机器学习/03-机器学习了解哪些算法|机器学习了解哪些算法]]
- [[八股/07-AI与Agent/01-Python与机器学习/00-Python与机器学习导航|Python与机器学习导航]]

---

## 06 来源与改写说明

题目线索来自 [goehou/agent_java_offer 仓库贡献者的搜索推荐资料](https://github.com/goehou/agent_java_offer/blob/298656dc4d0fb5f7db107fc6463f11230b3a49f7/docs/interview_prep/05_%E9%A1%B9%E7%9B%AE%E8%A1%A8%E8%BE%BE/04_%E6%90%9C%E7%B4%A2%E6%8E%A8%E8%8D%90%E5%B9%B3%E5%8F%B0/01_%E6%A0%B8%E5%BF%83%E9%97%AE%E7%AD%94.md)（[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)）。将源资料只列出的 SVD 拆成独立复习题，补充数学含义、检索流程及推荐边界。 原资料的社区面经或预测不作为具体公司必考题的事实依据，也不代表本人做过相关项目。

技术依据：

- [scikit-learn SVD 与 LSA](https://scikit-learn.org/stable/modules/decomposition.html#truncated-singular-value-decomposition-and-latent-semantic-analysis)
- [TruncatedSVD 的输入和符号约定](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.TruncatedSVD.html)
