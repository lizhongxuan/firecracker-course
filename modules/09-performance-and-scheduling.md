# 模块 09：性能、容量与训练调度

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/09-performance-and-scheduling.md) · [上一模块](08-harness-and-evaluation.md) · [下一模块](10-distributed-control-plane.md)

学习目标：从单实例性能走向端到端有效轨迹吞吐、排队与资源成本。

## 基础知识

延迟、吞吐、利用率和资源成本描述不同方面。用稳定系统中的平均到达率与停留时间估算平均并发，同时保留峰值、长尾和失败余量；等待模型的实例仍可能占据内存。

准备、恢复、首次请求、推理生成、工具、评测与提交各有队列。同步训练批次可能被长尾阻塞，异步采集则引入策略版本、背压与样本选择问题。

预热和快照要按实际负载比较，计入常驻资源、缓存未命中、私有页、重连和回收。全局 P95 可能掩盖较少的慢路径，必须同时报告条件分组、失败与样本数。

## 阅读资料

- [性能与容量专题](../references/observability-and-performance.md)
- [网络性能](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/network-performance.md)
- [快照性能测试](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/tests/integration_tests/performance/test_snapshot.py)
- [Agent Loop 管线](https://verl.readthedocs.io/en/latest/advance/agent_loop.html)
- [E2B 恢复与缓存架构](https://github.com/e2b-dev/runtime/blob/main/docs/ARCHITECTURE.md)
- [Kubernetes agent-sandbox 预热池](https://github.com/kubernetes-sigs/agent-sandbox)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：从单实例性能追到管线排队与资源账单。

1. 沿 verl 的 rollout 调度与结果汇集路径，定位并发任务如何等待推理和工具，找出批次完成与策略版本的关联。同步或异步行为以所选版本与配置为准。
2. 结合 E2B 恢复链，提出需要测量的镜像、缓存、缺页、首次工具响应与回收阶段；把文档中的设计解释转成待验证假设。
3. 阅读 agent-sandbox 的 SandboxWarmPool、SandboxClaim 与 SandboxTemplate 关系，分析池大小、补充速度、领取失败与闲置成本。Kubernetes 的真实实验在模块 10 进行，本模块先完成容量与排队模型。

## 实验与设计练习

1. 固定负载和版本，比较冷启动与恢复的不同计时点，记录样本、失败和冷热缓存条件。
2. 为 mock rollout 管线加入少量慢工具，观察不同队列和推理等待。
3. 用统一资源时间与有效轨迹口径比较预热方案，说明哪些优化只改变局部指标。
4. 给 mock 管线配置相同任务序列与少量慢工具，比较两个并发上限和两种预热策略；固定 seed、样本量及计时边界，记录准入、排队、有效轨迹数、失败与资源时间。
5. 将预热池耗尽、模板版本更换和策略权重更新加入同一负载回放，分别测量或推演其影响。模拟结果只能说明模型内行为，真实环境另测缓存命中、内存增长和恢复长尾。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

容量计算量纲正确，结论有对照证据；不通过丢弃困难/失败样本制造虚假吞吐。

项目验收：有一份跨阶段时间线、一张吞吐/成本对照表和一个预热池容量模型；按条件分组报告，说明调度改变是否引入样本选择偏差。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/09-performance-and-scheduling.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
