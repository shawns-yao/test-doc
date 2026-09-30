
LongMemEval 问答任务中 QA Accuracy 达 83.6%， Recall@5 达 95.8%

测什么，怎么算，对比谁，为什么有效


# 1.  83.6% QA Accuracy

首先是历史 会话按照时间顺序输入，工作记忆和长期记忆，收到 benchback ，然后是记忆检索

LongMemEval-S 当前 cleaned 数据集是 **500 道问题**。官方支持直接保存你的系统答案为 `question_id + hypothesis`，然后用它提供的 QA evaluation 脚本评分

也就是
按时间顺序把 LongMemEval 的历史 session 喂给 Agent，让我们的记忆模块正常执行写入、更新和检索。测试阶段只给模型问题，不给 ground truth evidence。
系统自己从长期记忆检索证据，再交给固定的 answer model 生成答案。最后使用 LongMemEval 官方 evaluation protocol 判定答案正确性。500 道问题中如果正确 418 道，就是 83.6%。



# 2.  95.8% Recall_all@5 又是怎么来的？
说明“绝大多数题需要的全部证据都能被找齐”，但那剩下的 4.2% 才是最值得研究的，因为它暴露的是记忆检索链路里的结构性失败。

因为 Recall_all 不是命中任意一条就算成功，而是需要的证据全部找齐才算成功。

# 3. 为什么 QA 只有 83.6%，Recall 却有 95.8%？

