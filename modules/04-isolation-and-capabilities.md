# 模块 04：资源隔离、工具权限与安全

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/04-isolation-and-capabilities.md) · [上一模块](03-lifecycle-and-long-tasks.md) · [下一模块](05-snapshots-and-fork.md)

学习目标：同时限制计算资源和工具能力，并验证宿主侧约束实际生效。

## 基础知识

namespace、降权、jailer、cgroup、seccomp 与宿主网络策略承担不同职责。vCPU 数不是物理核独占保证，客户机配置内存也不是全部宿主成本。

预算覆盖 CPU、内存、磁盘、网络、输出、时长和外部调用；日志采集器、工具代理及辅助进程也可能替任务消耗资源。

工具是否允许调用，取决于可信主体、动作、资源归属、参数和当前授权。白名单域名或 guest 不持有密钥不能代替具体业务权限；元数据中暴露给 guest 的秘密要按 guest 可读处理。

## 阅读资料

- [隔离与配额专题](../references/isolation-and-resources.md)
- [生产宿主机建议](../../../docs/prod-host-setup.md)
- [MMDS](../../../docs/mmds/mmds-user-guide.md)
- [cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html)
- [runc 配置与实现](https://github.com/opencontainers/runc)
- [gVisor 隔离边界](https://gvisor.dev/docs/architecture_guide/intro/)
- [E2B Runtime](https://github.com/e2b-dev/runtime)
- [OpenHands 安全与动作确认](https://docs.openhands.dev/sdk/guides/security)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：用项目实现核对资源、宿主隔离、动作确认与业务授权的边界。

1. 在 runc 的 OCI 配置处理和 Firecracker/jailer 配置中各找出一项资源约束；说明限制对象和配置实际生效位置。
2. 读 gVisor 的安全模型说明，与 Firecracker 的信任边界对照。对需要的系统调用与设备能力做兼容性检查，不从几条成功用例推导全面安全结论。
3. 在 E2B 中定位出站策略相关实现，在 OpenHands 中定位动作确认入口；检查执行时使用的主体、参数和策略版本。将网络许可、风险提示与业务资源授权分别记录。

## 实验与设计练习

1. 在自有测试实例上运行有界压力，分别记录 CPU 限流、内存事件和输出上限。
2. 使用本地 mock 代理测试允许、拒绝、改参数、撤权和越权主体路径。
3. 画出出站与工具代理流量，检查 guest 是否存在绕开宿主策略的通道。
4. 对同一组本地 mock 请求测试正常授权、参数变化、撤权与伪造主体；让高层动作确认已通过而实际业务授权失败，验证可信代理仍然拒绝执行。
5. 有合适 Linux 环境时，对普通容器与 gVisor 运行同一小型文件/网络负载，记录兼容性、资源用量和限制；先完成对比设计也可以，但需标注尚未实测。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

能够解释每层约束、给出超限证据，并区分运行时隔离与业务授权。对应实操 [E03](../expert-assessment/by-module/practical-exercises.md#e03)。

项目验收：为每项约束提供“配置 → 执行点 → 失败证据”，能说明框架动作确认与服务端授权之间仍需实现的合同。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/04-isolation-and-capabilities.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
