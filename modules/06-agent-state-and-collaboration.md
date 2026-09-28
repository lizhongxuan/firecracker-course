# 模块 06：多 Agent、记忆、技能与产物

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/06-agent-state-and-collaboration.md) · [上一模块](05-snapshots-and-fork.md) · [下一模块](07-rl-rollout-contracts.md)

学习目标：把协作、记忆、技能、工作区和产物建模为有归属和版本的状态。

## 基础知识

多 Agent 协作不仅是多个隔离进程。共享代码与消息需要基线、因果关系、冲突处理和结果提交语义，测试结论必须绑定实际被测产物。

对话、工作区、持久记忆、检索库和技能包有不同生命周期。技能声明的工具需求并不产生授权，委派能力应受父权限及总预算约束。

产物路径和内容仍来自不可信执行。可信端要控制对象归属、路径解析、类型和大小，同时按不可信内容展示；逻辑不可访问与介质物理清除是不同承诺。

## 阅读资料

- [工作区与存储专题](../references/storage-and-images.md)
- [平台接口专题](../references/execution-platform-design.md)
- [受约束路径解析](https://man7.org/linux/man-pages/man2/openat2.2.html)
- [OpenHands SDK 源码与包边界](https://github.com/OpenHands/software-agent-sdk)
- [OpenHands 架构](https://docs.openhands.dev/sdk/arch/overview)
- [OpenHands 持久化](https://docs.openhands.dev/sdk/guides/convo-persistence)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：主读 SDK、Tools、Workspace、Agent Server，追完一次动作与观测往返。

1. 在 `openhands-sdk` 中定位 Agent、Conversation、Event 与 Tool，记录动作如何进入执行器、观测怎样回到会话；在 `openhands-tools` 中选一个工具查看实现。
2. 对照 `openhands-workspace` 与 `openhands-agent-server`，标明 Agent、工具、文件与凭证实际在哪个进程和环境中。工具可能与 Agent 一起运行，不能假设所有工具调用都会通过 Workspace 发成远程 RPC。
3. 阅读技能、上下文和持久化相关入口，把提示/技能内容、会话状态、工作区、长期记忆的作用域分开。由自己设计父子权限、基线合并与预算合同，不默认 SDK 会完成课程全部隔离要求。

## 实验与设计练习

1. 用本地测试仓库模拟三条协作分支，绑定基线、修改与测试结果，处理冲突和取消后的草稿。
2. 设计父子 agent 的权限交集、总预算和取消树，使用虚构工具验证不能通过委派扩大权限。
3. 对产物服务测试越界、链接、超大文件和版本错误；材料不外发。
4. 使用固定的模型响应 fixture 和虚构工具，追踪“提出动作 → 执行 → 产生观测 → 写入会话事件”；保存每一步的任务、工具、工作区与产物版本。
5. 在三个独立本地测试工作区模拟接口修改、实现修改和测试；让一次测试针对旧基线完成，验证汇总结果能够识别版本不匹配。只有自制模拟器时标注未验证 OpenHands 的实际集成。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

交付状态归属表与协作时序，能证明私有状态不串扰、产物来源和测试版本可核查。

项目验收：交付 OpenHands 组件与实际执行位置图、动作/观测记录和协作版本证据；明确哪些能力来自项目，哪些是自己实现。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/06-agent-state-and-collaboration.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
