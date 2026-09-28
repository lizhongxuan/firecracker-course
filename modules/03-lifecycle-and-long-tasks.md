# 模块 03：生命周期、取消与长任务

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/03-lifecycle-and-long-tasks.md) · [上一模块](02-images-boot-and-io.md) · [下一模块](04-isolation-and-capabilities.md)

学习目标：把创建、执行、取消、恢复和清理组织成可对账的生命周期。

## 基础知识

请求超时可能是没执行，也可能是成功后丢了响应。稳定身份、幂等关系、持久意图和实际资源观察一起决定是否重试。

取消受理、任务停止、结果提交和资源清理是不同状态。并发完成与取消需要明确顺序，管理进程重启后仍要核对资源，而不能只相信一条数据库状态。

长任务跨越时间、版本和权限变化。检查点记录的是一个执行位置，当前授权、外部结果和已提交事件不能任意回滚到旧时刻。

## 阅读资料

- [生命周期专题](../references/api-and-lifecycle.md)
- [平台状态设计专题](../references/execution-platform-design.md)
- [Firecracker API](../../../src/firecracker/swagger/firecracker.yaml)
- [E2B 生命周期架构](https://github.com/e2b-dev/runtime/blob/main/docs/ARCHITECTURE.md)
- [OpenHands 会话持久化](https://docs.openhands.dev/sdk/guides/convo-persistence)
- [Python 异步任务与取消](https://docs.python.org/3/library/asyncio-task.html)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：E2B 追实例状态，OpenHands 追会话状态；对照课程中的 task/attempt/instance 模型。

1. 从 E2B `packages/api` 的生命周期入口进入 `packages/orchestrator`，选创建、停止或暂停中的一条链，标出存储写入、远程请求、失败返回和资源清理的位置。
2. 从 OpenHands 的 Conversation 和持久化文档定位会话状态与事件保存；列出它们和 VM、工具进程、外部业务结果分别由谁负责。
3. 为一个取消路径记录信号发出、等待退出、结果提交、清理失败的顺序。Python 适配层阅读协程取消与清理语义，不能把异步任务被取消等同于外部进程已经停止。

## 实验与设计练习

1. 定义任务、attempt 与实例身份，画正常、取消和失败状态机。
2. 在测试实例已启动但状态未保存的窗口中断自己的管理器，再恢复并对账。
3. 推演审批等待期间撤权、版本升级与完成/取消竞态，记录执行检查点和结果合同。
4. 参考读到的链路，为本地管理器增加一个故障注入点；在“资源已创建但状态未保存”后重启，保存对账前后的资源与状态清单。
5. 用固定动作 fixture 模拟会话持久化后恢复；让一条 mock 工具操作已完成但确认丢失，检查恢复是否重复产生副作用。将自己的恢复策略与所选项目版本的行为分开记录。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

重复创建/取消/清理有明确结果；资源有所有权，未知状态不被报告为成功。对应实操 [E01](../expert-assessment/by-module/practical-exercises.md#e01)。

项目验收：至少有一条源码状态链和一个可复现失败实验；会话、运行时和外部结果的恢复责任明确。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/03-lifecycle-and-long-tasks.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
