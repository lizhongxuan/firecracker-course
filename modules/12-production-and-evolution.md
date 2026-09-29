# 模块 12：技术选型、发布与生产演进

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/12-production-and-evolution.md) · [上一模块](11-engineering-and-source.md)

学习目标：用一致证据做技术选型、版本发布、维护与演进决策。

## 基础知识

选型比较相同工作负载、权限边界、兼容需求与成本口径，纳入 reset/fork、长会话、调试和维护。可以选择成熟服务或自研模块，关键是说明依据和改变选择的条件。

发布关联运行时、宿主/客户机、镜像、工具协议、技能和快照兼容条件。节点维护需要停止准入、容量预留、有界排空与失败处置；二进制回滚不能撤销所有状态变化。

工程经历以个人负责范围、实际变更、失败、验证和上线结果为证据。公开贡献是一种来源，私有项目可使用脱敏本地材料；不按项目名气替代研发能力。

## 阅读资料

- [生产配置依据](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/prod-host-setup.md)
- [版本策略](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/RELEASE_POLICY.md)
- [快照兼容性](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/snapshotting/versioning.md)
- [短任务平台参考案例](../references/capacity-and-state-case.md)
- [gVisor 架构对照](https://gvisor.dev/docs/architecture_guide/intro/)
- [containerd 版本与运行时关系](https://github.com/containerd/containerd)
- [E2B 运行平台](https://github.com/e2b-dev/runtime)
- [Kubernetes agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：收束前面项目证据，完成可被复核的选型与演进决策。

1. 在同一层比较普通容器/runc、gVisor、Firecracker 的执行边界；在平台层比较 E2B 与 Kubernetes agent-sandbox 的状态管理方式。OpenHands、Harbor、verl 分别参与执行、评测、训练衔接，不与 VMM 做替代关系排名。
2. 整理运行时、镜像、模板、Agent/工具、事件 schema、评测与训练适配层的版本矩阵；以一个变更追踪受影响对象、兼容测试与无法回滚的状态。
3. 从自己的课程实现选择一次可解释的修复或设计变化，用本地变更与测试说明责任、取舍和维护成本。公开开源贡献只是证据来源之一；本课程无需发布代码或提交 PR。

## 实验与设计练习

1. 对两个熟悉的运行时方案制定同条件 POC，写出成功/失败条件和未验证项。
2. 桌面演练宿主升级期间的长会话与 rollout 排空，处理无法跨版本恢复的任务。
3. 完成 [S01](../expert-assessment/by-module/system-design.md#s01) 综合设计，并用一次真实或脱敏工程经历说明决策与验证方法。
4. 用前面同一组文件/网络/长任务负载形成两种运行时的 POC 报告，明确硬件、版本、隔离策略、失败样本与成本；未实测项不填推测数字。
5. 为 [S01](../expert-assessment/by-module/system-design.md#s01) 选择实际需要的组件，画出完整执行与数据链；演练一次模板升级、一次事件协议变更和一次宿主维护，说明如何暂停推进、回退程序或处理不兼容会话。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

方案可落地、可测量、可演进；灰度、回滚和不可逆状态边界明确，个人贡献可解释。

项目验收：交付分层选型表、版本矩阵、[S01](../expert-assessment/by-module/system-design.md#s01) 架构及一项本地工程变更的验证记录；能根据新证据修改原来的选择。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/12-production-and-evolution.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
