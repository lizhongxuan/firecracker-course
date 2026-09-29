# 模块 11：Go/Rust 研发与源码验证

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/11-engineering-and-source.md) · [上一模块](10-distributed-control-plane.md) · [下一模块](12-production-and-evolution.md)

学习目标：选择 Go 或 Rust 交付可运行执行器，并能用源码解释后端行为。

## 基础知识

有界并发、取消传播、任务所有权、关闭顺序与错误状态是执行器的基本合同。取消信号不保证任意阻塞工作立即结束，需要明确后端契约与必要的进程边界。

源码阅读从一个行为或失败出发，跟踪解析、状态分发、资源构建、执行与错误传播。设备路径还需处理不可信缓冲区、在途 I/O、事件循环公平性和暂停/快照一致性。

AI Coding 的交付责任包括定义独立验收条件、审查依赖/API、验证并发和失败，以及现场解释和修改。编译、借用检查或 race detector 各有适用范围，不等于全部逻辑正确。

## 阅读资料

- [源码与事件循环专题](../references/source-and-event-loop.md)
- [Go 并发检查](https://go.dev/doc/articles/race_detector)
- [Tokio 优雅关闭](https://tokio.rs/tokio/topics/shutdown)
- [Firecracker 测试指南](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/tests/README.md)
- [E2B Go 实现](https://github.com/e2b-dev/runtime)
- [OpenHands Python SDK](https://github.com/OpenHands/software-agent-sdk)
- [verl 源码与接口](https://github.com/verl-project/verl)
- [Python 异步任务](https://docs.python.org/3/library/asyncio-task.html)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：Go/Rust 任选一门完成 worker；Python 掌握适配、异步与测试所需部分。

1. Go 路线从 E2B 的节点编排或 envd 选择一个请求生命周期，追踪上下文、并发、错误与清理。Rust 路线沿 Firecracker 的启动或设备路径，核查资源所有权与暂停/快照要求。声称精通 Firecracker 时仍需阅读其源码。
2. Python 学习范围为虚拟环境、类型/数据模型、JSON 与进程或 HTTP 接口、asyncio 取消和测试。修改一个 OpenHands/verl 适配层，说明异常与 deadline 如何跨到 Go/Rust worker。
3. 为跨语言协议约定 task/attempt、版本、截止时间、结束原因、输出上限与事件标识；核对取消、重试和后端关闭的实际行为。课程接口不假定与三个项目原生协议相同。

## 实验与设计练习

1. 沿熟悉运行时追踪一次启动和一次 I/O；声称精通 Firecracker 时使用本地 Firecracker 源码。
2. 选择 Go/Rust 完成 [E04](../expert-assessment/by-module/practical-exercises.md#e04) 的有界 worker、结束分类、输出限制与事件回放。
3. 现场改变后端延迟和取消时机，解释并验证唯一终态、资源释放与回放无新副作用。
4. 为 [E04](../expert-assessment/by-module/practical-exercises.md#e04) worker 编写最小 Python 客户端与固定 fixture，将一条 mock 轨迹送入 Go/Rust 执行器；验证字段兼容、未知版本、超长输出与超时。框架实际适配作为课程集成练习，不增加限时 [E04](../expert-assessment/by-module/practical-exercises.md#e04) 的必做范围。
5. 在本地完成一个小改动，例如取消清理、错误分类或事件关联；保存改动前失败、改动后通过的测试和自己的解释。审查 AI 生成代码与测试时，以独立契约验证而不是复述实现。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

交付源码、运行方法、有效测试和未覆盖范围；mock 验证与真实运行时证据分别记录。

项目验收：Go/Rust 可运行交付、Python 适配测试、至少一条项目源码链以及一个本地修复证据；逐项区分自制协议、框架适配和真实运行时。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/11-engineering-and-source.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
