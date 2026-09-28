# 模块 10：分布式控制面与副作用

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/10-distributed-control-plane.md) · [上一模块](09-performance-and-scheduling.md) · [下一模块](11-engineering-and-source.md)

学习目标：把节点与工具执行组织成可恢复控制面，明确外部副作用的保证边界。

## 基础知识

任务、attempt、实例和工具调用是不同身份。控制面保存期望状态，节点报告实际状态，对账修复差异；租约到期不证明旧进程停止。

执行代次和可信端校验可以拒绝旧结果，但外部服务如果没有幂等或事务支持，仍可能出现结果不确定。消息确认和分布式锁不会凭空消除外部操作的提交窗口。

授权链绑定可信用户、任务、工具版本、参数和资源归属。恢复旧状态、参数变化和撤权需要重新判断当前执行权，已产生的副作用不能靠删除实例撤销。

## 阅读资料

- [控制面与任务协议专题](../references/execution-platform-design.md)
- [生命周期专题](../references/api-and-lifecycle.md)
- [工具代理与令牌边界](https://modelcontextprotocol.io/docs/draft/tutorials/security/security_best_practices)
- [E2B 控制面与节点编排](https://github.com/e2b-dev/runtime/blob/main/docs/ARCHITECTURE.md)
- [Kubernetes agent-sandbox 控制器](https://github.com/kubernetes-sigs/agent-sandbox)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：到此模块再深入集群控制；先理解 Pod、CRD、期望/实际状态、控制器对账与资源所有权。

1. 对照 E2B 的 API/节点编排职责与 agent-sandbox 的 Sandbox/Claim/Template/WarmPool，画出对象归属和请求链。两者作为不同控制面方案阅读，不默认要在同一系统叠加部署。
2. 在 agent-sandbox 所选版本中搜索 SandboxClaim 和 Reconcile，追踪一次领取、资源创建、状态更新与清理；核对并发冲突、重复事件和控制器重启的处理。引用具体位置，而非假设所有控制器都有相同保证。
3. 对每条控制链补上“旧执行者仍存活”的时序，标出执行代次、结果提交与外部工具之间的保证缺口。控制器对账不会自动解决业务副作用的幂等问题。

## 实验与设计练习

1. 用本地两个模拟节点演示旧执行者仍存活、新代次开始执行时的结果提交。
2. 模拟外部 mock 已执行但结果确认丢失，分别讨论可查询/幂等和不支持这些机制的合同。
3. 记录授权到实际执行的关联链，验证旧参数、旧代次和撤权后的新请求被正确处理。
4. 在已有、允许实验的 Kubernetes 环境中，创建小型测试预热池和两个 Claim，验证领取所有权、重复对账、控制器重启和删除后的资源回收；不把别人的共享集群当实验环境。
5. 没有集群时用本地状态存储和两个模拟控制器复现 Claim 竞争，提交事件时序及冲突处理伪代码；记录该结果尚未验证 Kubernetes API、真实 Pod 与网络分区。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

画出至少两个不确定结果窗口；保证与依赖外部服务的条件明确，不能笼统承诺恰好执行一次。

项目验收：交付一次领取/恢复的源码路径、所有权与对账时序、至少一个控制器失败场景；每项证据标注真实集群或模拟。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/10-distributed-control-plane.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
