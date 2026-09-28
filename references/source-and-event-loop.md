# 专题：Rust 源码、事件循环与 KVM 执行

[统一课程](../README.md) · [面试题与答案](../expert-assessment/by-module/README.md)

这是按主题组织的阅读与实验资料；课程模块及考核以总目录为准。


前置：理解运行时、生命周期、隔离和快照；会读基本程序控制流。目标：跟踪两个具体行为，不要求逐行读完整个仓库。

## 1. 先学基础

**Rust 阅读最小集。**所有权与借用帮助理解资源何时释放；`Result` 和 `?` 解释错误如何传播；`enum` 与 `match` 表达动作和状态；`Arc`、`Mutex` 与 channel 表达共享和线程通信；`unsafe` 边界需要额外检查不变量。先在实际代码中辨认这些结构，再补语言细节。可读 [The Rust Programming Language](https://doc.rust-lang.org/book/)，优先所有权、错误处理与并发章节。

**事件驱动。**`epoll` 等机制让线程等待多个事件源，而不是给每个设备请求都创建线程。事件处理器若长时间阻塞，可能拖慢同一循环中的其他工作；事件驱动也需要考虑公平、背压和错误处理。

**KVM_RUN 与退出。**vCPU 线程通过 KVM 运行客户机；需要用户态处理的退出会交回 VMM，例如某些 I/O 或 MMIO。不要把客户机每条指令都理解为由 Firecracker 逐条模拟。[KVM API](https://docs.kernel.org/virt/kvm/api.html)

**virtio 队列。**客户机通过描述符链描述缓冲区，设备后端消费请求并报告完成。客户机提供的地址和长度跨越信任边界，需要检查；具体通知路径由设备与传输方式决定。

## 2. 两条源码阅读路线

| 路线 | 文件 | 需要回答 |
| --- | --- | --- |
| 启动入口 | [main.rs](../../../src/firecracker/src/main.rs) | 程序入口如何组织 API 与运行流程？ |
| 动作解析 | [actions.rs](../../../src/firecracker/src/api_server/request/actions.rs) | `InstanceStart` 如何变成 `VmmAction::StartMicroVm`？ |
| API 适配 | [api_server_adapter.rs](../../../src/firecracker/src/api_server_adapter.rs) | 运行期 API 与 VMM 如何交互？预启动路径有何不同？ |
| 状态分发 | [rpc_interface.rs](../../../src/vmm/src/rpc_interface.rs) | `handle_preboot_request` 如何检查动作与状态？ |
| 资源创建 | [builder.rs](../../../src/vmm/src/builder.rs) | `build_microvm_for_boot` 如何组织内存、KVM 和设备？ |
| vCPU 执行 | [vcpu.rs](../../../src/vmm/src/vstate/vcpu.rs) | `run_emulation` 如何处理运行与退出？ |
| 网络 I/O | [net/event_handler.rs](../../../src/vmm/src/devices/virtio/net/event_handler.rs) | 哪些事件推动收发和限流？ |
| virtio 队列 | [queue.rs](../../../src/vmm/src/devices/virtio/queue.rs) | 描述符如何校验与消费？ |

这些是导航入口，不意味着每个文件都是同一次调用的直接上下级。结合 [Design：Internal Architecture](../../../docs/design.md) 区分 API、VMM 与 vCPU 线程。

## 3. 实验 A：追踪一次启动

在 Mac 本地仓库根目录阅读，不需要 KVM：

```bash
rg -n 'InstanceStart|StartMicroVm' src/firecracker/src src/vmm/src
rg -n 'handle_preboot_request|start_microvm' src/vmm/src/rpc_interface.rs
rg -n 'build_microvm_for_boot|resume_vm' src/vmm/src
rg -n 'run_emulation|KVM_RUN' src/vmm/src/vstate
```

从 API 解析出发，记录“输入类型 → 转换后的动作 → 状态校验 → 资源构建 → 开始执行 → 响应或错误”。给每一步标注文件与函数，而不是只抄文件列表。找一个缺失内核配置错误，追踪它如何传回 API。

## 4. 实验 B：追踪一次 I/O

选网络发送或块设备读取中的一个，画出客户机队列、通知事件、事件处理器、宿主 I/O、完成通知的关系。标出缓冲区所有权、长度检查、错误处理和限流位置。区分控制配置 API 与设备数据路径；业务数据不会逐包经过 HTTP API。

选做：在合适的 Linux 开发环境构建同一源码版本，并按 [测试指南](../../../tests/README.md) 运行与阅读内容相关的一项测试。完整集成测试依赖 KVM 和对应资源，不要求在 macOS 上通过它们。若添加调试日志，只做局部修改，并说明测量结果可能被日志扰动。

## 5. 验收

提交两张调用/事件图：启动、I/O。每张图至少标注一个错误边界和一个跨线程或跨宿主/客户机边界。能够指出源码证据，并解释 API 配置路径为什么不同于 I/O 快速路径。
