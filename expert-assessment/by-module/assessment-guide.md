# 考核说明、统一权重与核验标准

[面试文档目录](README.md) · [实操题](practical-exercises.md) · [综合设计](system-design.md) · [评分表](scorecard.md)

题目、答案和评分材料统一维护在本目录。12 份模块文档共 60 道主问题与 60 道带答案的深入追问，另有 4 项实操与 1 道综合设计题。

材料只保存本地，没有联系候选人或操作服务器。课程支持 MicroVM/容器方案和 Go/Rust 二选一；额外声称精通 Firecracker 时，记录对应源码与真实 microVM 证据。学历等基本条件单独核验，开源贡献可作经验加分证据，不是替代核心能力的门槛。

## 项目证据的使用

课程以 E2B、OpenHands、verl 为主要源码案例，穿插 containerd/runc、gVisor、Harbor 与 Kubernetes agent-sandbox。它们是学习与实现参考，不是招聘方指定技术栈；候选人可以用同层、同等深度的实现证明能力。

沿现有题号核验三个方面：是否理解组件边界，能否指出源码和版本依据，是否有对应失败场景的实验或可验证设计。未使用过某个指定项目不自动扣分；只报项目名也不加分。一次证据可以支持多个不同能力点，但每题仍按原有标准评分。

E04 的限时必考范围仍为本地 mock worker。课程中的 Python 适配、Harbor 用例和集群实验用于学习与扩展验证，不要求在 E04 的 120 分钟内全部实现。题量、题号与总分仍为 60 题、4 项实操、1 道综合设计和 100 分。

## 模块与权重

| 模块 | 题号 | 题数 | 理论权重 |
| --- | --- | --- | --- |
| 01 运行时基础与架构边界 | Q01-01～Q01-04 | 4 | 3 |
| 02 镜像、启动与网络存储 | Q02-01～Q02-06 | 6 | 4 |
| 03 生命周期、取消与长任务 | Q03-01～Q03-04 | 4 | 5 |
| 04 资源隔离、工具权限与安全 | Q04-01～Q04-06 | 6 | 6 |
| 05 快照、恢复与 fork | Q05-01～Q05-06 | 6 | 6 |
| 06 多 Agent、记忆、技能与产物 | Q06-01～Q06-06 | 6 | 5 |
| 07 RL Rollout 与训练数据契约 | Q07-01～Q07-04 | 4 | 7 |
| 08 Harness、确定性与评测 | Q08-01～Q08-05 | 5 | 7 |
| 09 性能、容量与训练调度 | Q09-01～Q09-05 | 5 | 5 |
| 10 分布式控制面与副作用 | Q10-01～Q10-04 | 4 | 5 |
| 11 Go/Rust 研发与源码验证 | Q11-01～Q11-06 | 6 | 4 |
| 12 技术选型、发布与生产演进 | Q12-01～Q12-04 | 4 | 3 |

## 统一计分

理论共 60 分，每题原始分为 0–4；模块得分 = 模块权重 × 该模块原始得分之和 ÷（4 × 题数）。实操 E04 必考 20 分，E01–E03 选一项 10 分，综合 S01 为 10 分，总分 100。重复完成选考实操不额外加分。

0 分：核心机制错误或无法回答；1 分：仅列术语；2 分：机制与正常流程成立；3 分：能处理失败、比较取舍并设计验证；4 分：还有可核查依据并能应对条件变化。未问到或环境不具备标“未验证”，不靠已问题目的平均分推算整卷通过。

建议强匹配参考总分 85 以上，同时理论至少 45/60、模块 07+08 至少 10/14、E04 至少 14/20、选考实操至少 7/10、S01 至少 7/10，并无未澄清的关键安全或一致性误解。这是本考核的建议，不是招聘方的官方标准。岗位适配与 Firecracker 专长的核验结论分别陈述，共用证据，不建立第二套总分。

## 模块精通判定

分模块文档沿用现有 60 道题，每题加入一道有答案的条件变化追问。建议全部题目覆盖、每题至少 3 分、至少两题 4 分，并通过该模块现场核验且无未澄清关键误解时，记录“本模块已展示精通”。仅抽查或缺少证据时记录“已覆盖部分达标/待验证”。这是同一考核的模块结论，不另算总分，也不能替代生产经验核验。

## 面试安排

第一轮 90 分钟筛查：10 分钟项目经历，20 分钟运行时/隔离/生命周期，25 分钟 RL/Harness，20 分钟 fork/协作/分布式状态，10 分钟研发方法，5 分钟记录缺口。建议题号为 Q01-03、Q03-01、Q04-03、Q05-05、Q06-01、Q07-01、Q07-02、Q08-01、Q08-02、Q10-02、Q11-05；根据回答选择追问，不要求全部问完。

第二轮完成 E04：有 mock 骨架时建议 120 分钟，无骨架则增加时间或缩小范围。再选择 E01/E02/E03 做对应验证，单独安排 S01 架构讨论。60 题是完整题库，可分多轮或用于书面准备；筛查结论只是进入后续验证的依据。

允许查文档和在本地材料约束下使用辅助工具。候选人必须现场解释关键路径、完成一次条件变更并提供测试。不要用背默认值代替专家判断，也不要把 mock 通过当作真实运行时证明。

## 作答与核验

每题说明假设、机制、失败处理、验证证据和代价；引用资料时标明版本。参考答案是一条可行思路，按需求、版本和威胁模型判断，接受论证完整的其他方案。澄清后自行修正时，按最终完整表现评估。

声称精通 Firecracker 时，应核验模块 01 的调用路径与环境判定、模块 05 的快照机制、模块 11 的实际源码路径，并提供真实 microVM 证据。Q11-01/Q11-02 对容器方向候选人可使用其熟悉运行时的等价路径，但应单独记录未验证的 Firecracker 专长。

所有数字为教学题设。实验仅使用本地考核环境、虚构凭证与 mock 业务，文件与日志留在本地；要求真实运行时证据的项目，按题目使用预先准备的可信测试环境。源码推理、mock、真实运行时及生产经验分别记录。

## 关键误解

澄清后仍认为 microVM 自动保证工具授权、旧审批可在撤权后重放、快照自动保存外部事务，或队列可让任意外部操作恰好一次，不能用其他题的高分掩盖。对于训练场景，静默把基础设施失败当策略结果、允许任务篡改可信评分、无条件把回放当成真实确定执行，也必须纠正并验证。

保留原话、追问和修正证据；不对未考察领域或个人诚信下结论。

## 技术核验入口

Firecracker 参考本地提交 `30471852666564d980f330d0575115eda7d5ce8e`；其他版本先核对差异。以下链接用于核验技术机制，评分规则与上层架构是本考核自行设计。

| 主题 | 一手依据 |
| --- | --- |
| 架构、启动与设备 | [Design](../../../../docs/design.md)、[API](../../../../src/firecracker/swagger/firecracker.yaml) |
| 调用链 | [actions.rs](../../../../src/firecracker/src/api_server/request/actions.rs)、[rpc_interface.rs](../../../../src/vmm/src/rpc_interface.rs)、[builder.rs](../../../../src/vmm/src/builder.rs)、[vcpu.rs](../../../../src/vmm/src/vstate/vcpu.rs) |
| 隔离与配额 | [jailer](../../../../docs/jailer.md)、[生产宿主机建议](../../../../docs/prod-host-setup.md)、[Linux cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html) |
| 元数据与通信 | [MMDS](../../../../docs/mmds/mmds-user-guide.md)、[vsock](../../../../docs/vsock.md)、[网络](../../../../docs/network-setup.md) |
| 快照与磁盘 | [Snapshot Support](../../../../docs/snapshotting/snapshot-support.md)、[Versioning](../../../../docs/snapshotting/versioning.md)、[克隆随机状态](../../../../docs/snapshotting/random-for-clones.md)、[块缓存](../../../../docs/api_requests/block-caching.md) |
| KVM 边界 | [Linux KVM API](https://docs.kernel.org/virt/kvm/api.html) |
| 路径受约束解析 | [Linux openat2 手册](https://man7.org/linux/man-pages/man2/openat2.2.html)；它不能代替应用级授权与内容来源检查 |
| 代理与令牌安全 | [MCP 官方安全说明](https://modelcontextprotocol.io/docs/draft/tutorials/security/security_best_practices)：核对 audience、token passthrough、状态与主体绑定、SSRF；按具体协议版本判断 |

资料在线版本可能变化；不把某一版本的默认值当作跨版本事实，也不要求候选人用本手册相同术语才能得分。

RL/Harness 另见 [Gymnasium Env](https://gymnasium.farama.org/api/env/)、[verl Agent Loop](https://verl.readthedocs.io/en/latest/advance/agent_loop.html)；语言并发验证见 [Go race detector](https://go.dev/doc/articles/race_detector) 与 [Tokio shutdown](https://tokio.rs/tokio/topics/shutdown)。接口和默认值按选定版本核对。


项目机制另见 [E2B 架构](https://github.com/e2b-dev/runtime/blob/main/docs/ARCHITECTURE.md)、[OpenHands 架构](https://docs.openhands.dev/sdk/arch/overview)、[Harbor 任务与评测](https://docs.harborframework.com/core-concepts/tasks/overview)、[Kubernetes agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox)、[containerd](https://github.com/containerd/containerd)、[runc](https://github.com/opencontainers/runc) 与 [gVisor 架构](https://gvisor.dev/docs/architecture_guide/intro/)。项目能力按候选人声明的版本核验；题面中的故障与平台保证是考核要求，不能默认这些项目已实现全部要求。
