# 模块 02：镜像、启动与网络存储

[课程总目录](../README.md) · [本模块面试与答案](../expert-assessment/by-module/02-images-boot-and-io.md) · [上一模块](01-runtime-foundations.md) · [下一模块](03-lifecycle-and-long-tasks.md)

学习目标：理解内核/镜像、启动、网络和磁盘路径，能逐层定位任务环境不可用。

## 基础知识

内核负责驱动和系统资源，rootfs 提供用户态程序及依赖，PID 1 组织用户空间启动。OCI 镜像与块设备 rootfs 有不同格式和运行入口，构建/转换过程也需要隔离。

进程存在、API 可用、客户机启动、agent 握手、应用就绪是不同检查点。动态解释器缺失可能导致文件存在但无法执行，定位时需要具体错误与文件格式证据。

TAP、路由、NAT 和访问控制职责不同；vsock 是宿主与客户机通信能力。块设备把客户机 I/O 接到宿主文件或设备，缓存与 flush 决定持久性语义。

## 阅读资料

- [内核与启动专题](../references/kernel-rootfs-and-boot.md)
- [网络与 vsock 专题](../references/network-and-vsock.md)
- [存储与镜像专题](../references/storage-and-images.md)
- [官方入门](../../../docs/getting-started.md)
- [E2B 架构与组件入口](https://github.com/e2b-dev/runtime/blob/main/docs/ARCHITECTURE.md)
- [containerd 镜像与存储](https://github.com/containerd/containerd)
- [runc OCI bundle 示例](https://github.com/opencontainers/runc)

先读本模块基础说明，再带着题目查看对应资料。官方接口按实验选择的版本核对；平台层设计不默认由 Firecracker 原生实现。

## 项目源码阅读

深度：追踪镜像到运行环境、客户端到进程输出两条链。

1. 沿 containerd 的镜像拉取、解包与 snapshotter 概念阅读，再对照 runc bundle 的 rootfs 与配置。标注 OCI 镜像、可挂载文件系统、VM 根块设备之间需要的转换。
2. 从 E2B 架构文档定位 `packages/orchestrator` 中的模板构建职责和 `packages/envd` 的进程/文件接口。追踪一项启动配置如何到达运行环境，以及退出码与输出如何返回。
3. 为控制 API、沙箱流量和客户机进程各画一条路径，记录超时、就绪与错误由哪层报告。先完成源码追踪，整套 E2B 部署留到环境条件满足时。

## 实验与设计练习

1. 在 Linux 用同架构的可信内核与 rootfs 启动测试实例，记录不同就绪检查点。
2. 分别制造一项启动配置故障和一项应用监听故障，用串口/API/网络证据区分失败层。
3. 为两个实例准备独立可写数据空间，验证写入隔离与按合同保留的数据。
4. 对可信容器镜像与 Firecracker rootfs 各列一份启动清单，比较架构、入口、用户、工作目录与可写层；验证镜像中有文件不等于目标运行环境可执行。
5. 在本地 mock 客户机代理中加入 stdout/stderr 分流、退出码与输出上限，设计与 E2B envd 对应的请求时序；清楚标明哪些接口是自己的教学协议。

真实运行时实验在 Linux 执行；Mac 用于阅读、绘图与本地 mock。只操作自己的测试资源，材料保存在本地。本次交付课程文档，没有执行服务器实验。

## 验收

交付启动时间线、请求数据路径与磁盘关系图；能按证据修复故障，不通过清空宿主配置试错。

项目验收：镜像转换图、E2B 组件调用链及 mock 协议记录齐全；能够区分文件系统快照与包含内存/CPU 状态的 VM 快照。

## 面试与自测

完成学习与实验后，打开 [本模块面试题与答案](../expert-assessment/by-module/02-images-boot-and-io.md) 自测。题目、参考答案、追问和评分记录统一维护在 `by-module/`。
