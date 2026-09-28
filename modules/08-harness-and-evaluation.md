# 模块 08：Harness、确定性与评测

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/08-harness-and-evaluation.md) · [上一模块](07-rl-rollout-contracts.md) · [下一模块](09-performance-and-scheduling.md)

学习目标：设计可观察、可回放、可评估的 Harness，并明确确定性保证的范围。

## 基础知识

Harness 把模型、环境、工具和评分器组织成执行协议。事件包含主体、任务/尝试/步骤、来源、版本与因果关系；时间戳本身不能解决跨机器乱序、重复和缺失。

确定性需要定义粒度。固定 seed 不会固定所有网络、时间、并发和外部状态；录制工具结果回放、真实重新执行、重新采样模型是不同能力。回放要能检测输入或版本分歧，不能无条件返回旧成功。

可信评测需要保护评分逻辑和结果来源，任务代码不能任意修改其奖励。比较版本时要同时记录任务与策略版本、预算、重试、缺失和失败样本，区分环境改善与模型能力变化。

## 阅读资料

- [观察与测量专题](../references/observability-and-performance.md)
- [verl 多轮轨迹接口](https://verl.readthedocs.io/en/latest/advance/agent_loop.html)
- [环境结束语义](https://gymnasium.farama.org/api/env/)
- [OpenHands 架构与事件](https://docs.openhands.dev/sdk/arch/overview)
- [Harbor 任务结构](https://docs.harborframework.com/core-concepts/tasks/overview)
- [Harbor Verifier](https://docs.harborframework.com/core-concepts/tasks/verifier)
- [Harbor 独立验证环境](https://docs.harborframework.com/core-concepts/tasks/separate-verifier)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：OpenHands 用于追踪执行事件；Harbor 用于阅读任务、试验和评分过程。

1. 沿 OpenHands 的动作/观测事件链与持久会话记录，标注事件来源、顺序、缺失与恢复位置；设计自己的跨组件关联字段。
2. 阅读 Harbor 的任务配置、environment、solution 和 tests/verifier，区分任务输入、参考解与评分逻辑；追踪一次 trial 如何产生结果，并记录框架异常如何报告。
3. 比较评分与任务在同一环境运行、使用独立验证环境两种布置。独立环境仍需核对产物来源、挂载、凭证、结果写权限和版本；将这些条件作为验证目标。

## 实验与设计练习

1. 为 mock 工具录制动作、观测和事件，回放时禁止再次调用后端；改一个动作或版本应报告首个分歧。
2. 注入重复、乱序和迟到事件，验证终态不会被旧执行覆盖。
3. 在独立授权边界运行虚构评分器，用负向样例验证任务无法直接改写可信分数。
4. 在本地创建三个小型 Harbor 格式任务：可确定成功、可确定任务失败、可注入基础设施错误。使用固定动作脚本或本地适配器完成试验，分别核对奖励与错误记录；任务与结果保留在本地。
5. 把 OpenHands 风格动作/观测事件映射到自己的统一事件协议，并与评测试验 ID 关联。录制后回放应禁止新工具副作用；再修改工作区评分文件或伪造产物来源，验证所设计的可信评分边界能发现问题。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

交付事件 schema、回放边界和评测可信链；数据和失败口径可以被第三人复核。对应 [E04](../expert-assessment/by-module/practical-exercises.md#e04) 的回放部分。

项目验收：交付三个本地评测用例、事件映射、首个回放分歧及评分权限证据；区分 Agent 执行框架与评测框架各自负责的状态。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/08-harness-and-evaluation.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
