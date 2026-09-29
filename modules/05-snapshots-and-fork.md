# 模块 05：快照、恢复与 fork

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/05-snapshots-and-fork.md) · [上一模块](04-isolation-and-capabilities.md) · [下一模块](06-agent-state-and-collaboration.md)

学习目标：定义完整检查点和分支 fork，说明恢复正确性、兼容性与成本。

## 基础知识

Firecracker 的内存与 VM 状态快照依赖外部磁盘和宿主资源。应用一致性、在途 I/O、网络重连与外部事务需要单独处理；加载 API 成功只是部分证据。

fork 在平台层应定义父检查点、分支身份、共享不可变基础、私有写入、独立能力和随机流。它不等于对运行中的 VMM 简单调用宿主 fork，也不自动复制所有业务状态。

快照可以节省初始化工作，但存在按需缺页、私有页增长、兼容矩阵和模板引用成本。获胜分支的产物可以成为新基线，不能通用地合并任意进程与外部副作用。

## 阅读资料

- [快照与恢复专题](../references/snapshots-and-recovery.md)
- [Snapshot Support](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/snapshotting/snapshot-support.md)
- [快照版本](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/snapshotting/versioning.md)
- [克隆随机状态](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/snapshotting/random-for-clones.md)
- [E2B 快照、恢复与 fork](https://github.com/e2b-dev/runtime)
- [E2B 组件与状态流](https://github.com/e2b-dev/runtime/blob/main/docs/ARCHITECTURE.md)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：对照 Firecracker 的原生快照接口与 E2B 平台实现，追踪一次恢复或分支创建。

1. 在 E2B `packages/orchestrator` 中按所选版本寻找快照、恢复与 fork 入口，记录其调用 Firecracker 的位置，以及内存、磁盘和身份信息的来源。
2. 对照平台的按需内存恢复、可写磁盘层和缓存设计，标出共享数据、私有数据、引用与回收责任。通过源码或实验核实共享程度，不能从 fork 名称推断复制成本。
3. 列出 VM 快照之外的会话、工具、授权与评分状态。复用平台的 VM fork 后，仍需由上层定义 Agent 分支与外部副作用合同。

## 实验与设计练习

1. 在同一兼容 Linux 环境创建可信检查点，记录内存、状态、磁盘和版本清单。
2. 恢复两个实例，分别建立身份、私有写入和 mock 权限；取消一个实例，检查另一个与父模板。
3. 分别测 API 恢复完成和首次应用就绪；解释哪些分支状态仍由控制面管理。
4. 将现有两个实例实验整理成教学接口 Fork(checkpoint, branch_spec)，输出父检查点与分支清单；接口可由自己的管理器实现，不能把它记为 Firecracker 原生 API。
5. 让两个分支写入不同内容并取消其中一个；记录模板引用、磁盘占用和可观测内存变化，再与 [Q05-06](../expert-assessment/by-module/05-snapshots-and-fork.md#q05-06) 的理想化页面模型比较。真实 E2B 后端未运行时只报告阅读结论。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

提供两个实例独立且可恢复的证据，说明快照外部状态、限制与回收边界。对应实操 [E02](../expert-assessment/by-module/practical-exercises.md#e02)。

项目验收：区分 Firecracker API、平台实现与上层 Agent 合同；给出父子资源清单、分支回收证据及成本模型中的遗漏项。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/05-snapshots-and-fork.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
