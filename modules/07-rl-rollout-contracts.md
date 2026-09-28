# 模块 07：RL Rollout 与训练数据契约

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/07-rl-rollout-contracts.md) · [上一模块](06-agent-state-and-collaboration.md) · [下一模块](08-harness-and-evaluation.md)

学习目标：理解 RL 基本闭环，并为训练提供可解释、可追溯的 rollout 数据。

## 基础知识

RL 中策略根据观测选择动作，环境产生后续观测与反馈，训练器用轨迹与奖励更新策略。episode、step、reset 与结束条件构成环境契约；执行环境负责状态和动作效果，不能替训练方静默决定数据语义。

正常终止、预算截断和基础设施失败需要保留区别。基础设施失败直接作为零奖励或静默丢弃都可能影响训练数据，处理方式要与算法和任务定义约定。

轨迹关联任务/分支、策略和环境版本、实际动作/观测、奖励来源与重试。推理侧保留训练所需的原始生成记录；同一前缀分出的结果有相关性，按最快或最高分筛选会改变样本分布。

## 阅读资料

- [Gymnasium 环境契约](https://gymnasium.farama.org/api/env/)
- [verl Agent Loop](https://verl.readthedocs.io/en/latest/advance/agent_loop.html)
- [本地执行器专题](../references/execution-platform-design.md)
- [verl 源码](https://github.com/verl-project/verl)
- [Python 协程与任务](https://docs.python.org/3/library/asyncio-task.html)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：优先读环境交互与轨迹输出，训练算法先理解策略、奖励与更新的基本关系。

1. 从 verl Agent Loop 文档进入所选版本源码，定位 AgentLoopBase/Output 与 Manager/Worker，或版本中的对应接口。追踪一次模型生成、一次工具调用及轨迹返回；文档与实现不一致时记录差异。
2. 检查原始 token、工具观测区间、训练 mask 和采样参数如何传递；按所选算法明确还需要哪些概率与策略版本信息，以及它们来自推理侧还是训练侧。
3. 对照 Gymnasium 的 reset/step 语义，为课程 worker 映射正常终止、预算截断与基础设施错误。课程的 Reset/Step/Close 只是教学合同，不能原样假定为 verl 的原生 API。

## 实验与设计练习

1. 实现或设计本地 mock 环境的 Reset/Step/Close，把正常结束、步数上限和模拟宿主失败分成不同记录。
2. 交替与并行运行不同身份的 episode，验证 reset 不残留私有状态。
3. 给同一前缀生成多个虚构分支，输出包含筛选、未完成和失败信息的轨迹清单。
4. 用固定生成 fixture 实现一个最小轨迹适配器：两轮生成中夹一次 mock 工具调用，验证原始 token 顺序、观测区间与 mask 对齐；保存策略/环境版本和采样条件。
5. 分别注入工具超时、reset 失败、重复响应与步数上限；输出可解释的结束原因和尝试记录。用测试比较“保留原始生成”与“重建聊天记录”两种路径，指出哪些条件下无法保证一致。无 GPU 阶段只验证数据与接口，不声称训练已收敛。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

解释环境、Harness、推理与训练的分工；不能把重试和环境失败伪装成一次正常策略转移。

项目验收：有一条 verl 调用链、一份教学接口到项目接口的映射和轨迹 fixture 测试；能说明环境错误是否进入训练样本及其决策依据。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/07-rl-rollout-contracts.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
