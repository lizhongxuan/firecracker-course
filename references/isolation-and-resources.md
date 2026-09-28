# 专题：jailer、隔离与资源治理

[统一课程](../README.md) · [面试题与答案](../expert-assessment/by-module/README.md)

这是按主题组织的阅读与实验资料；课程模块及考核以总目录为准。


前置：理解宿主/客户机边界、网络与磁盘。目标：解释各层限制的作用，并通过可信压力任务验证配额。

## 1. 先学基础

**最小权限。**进程只获得工作所需的身份、文件和系统调用能力。一个接口或目录可写，就可能影响执行内容；因此管理面、镜像目录和 socket 权限也属于隔离设计。

**namespace 与 chroot。**namespace 隔离部分系统资源视图，例如网络、进程编号；chroot 改变进程看到的文件系统根目录。它们各有边界，不能单独当作完整的安全方案。

**cgroup。**cgroup 管理一组进程的资源使用。vCPU 数是客户机看到的虚拟处理器数量，不等于宿主机保证的 CPU 时间；客户机配置内存也不等于宿主机总成本。宿主侧还需容纳 VMM、页表、I/O 等开销，并测量 cgroup 的实际计费情况。

**seccomp。**它限制宿主 Firecracker 线程可使用的系统调用，和客户机内部的系统调用处理属于不同层。资源配额解决资源争用，seccomp 缩小系统调用攻击面，两者不能互相替代。

## 2. 资料阅读

- 必读：[Jailer](../../../docs/jailer.md)，关注 UID/GID、chroot、cgroup、可选 network namespace 与路径处理。
- 必读：[生产宿主机建议](../../../docs/prod-host-setup.md)，关注默认 seccomp、宿主补丁、权限和输出限制。
- 必读：[Seccomp](../../../docs/seccomp.md) 与 [Design：Threat Containment](../../../docs/design.md)。
- 补基础：[Linux cgroup v2 官方文档](https://docs.kernel.org/admin-guide/cgroup-v2.html)，查 `cpu.max`、`memory.max`、`memory.events`；先确认服务器是否使用 v2。

## 3. Firecracker 中的对应关系

| 机制 | 主要职责 | 不能由它独自保证的事情 |
| --- | --- | --- |
| KVM 与硬件虚拟化 | 客户机执行与内存隔离基础 | 应用级鉴权和租户配额 |
| jailer 与降权 | 为 Firecracker 建立受限运行环境 | 自动配置全部网络策略 |
| cgroup | CPU、内存等资源治理 | 文件或网络访问授权 |
| seccomp | 限制宿主线程系统调用 | 业务数据隔离和凭证管理 |
| 宿主网络策略 | 约束入站、出站与租户互通 | 客户机应用协议的正确性 |

生产环境应按官方建议使用 jailer 或等效的严格约束。jailer 的网络 namespace 等行为需要明确配置；不能仅看到进程名就认定隔离完成。[官方宿主机建议](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md)

## 4. 实验：验证而非只填写配额

1. 按同版本文档，用匹配版本的 jailer 和静态构建 Firecracker 启动一台测试 VM。检查其 UID/GID、根目录和实际 cgroup 归属。
2. 配置测试实例 CPU 限制，运行有时间上限的可信 CPU 压力任务，对照宿主 CPU 使用与限流统计。
3. 为测试实例设置合理内存边界，留出 VMM 与宿主余量。逐步增加客户机内存压力，区分客户机自身 OOM 与宿主 cgroup OOM 的日志和事件。
4. 为输出设置大小上限，运行一个固定时长、固定速率的日志任务，确认不会无限增长。
5. 验证实例只能访问预期的数据盘和网络目标；清理后检查配额目录及相关资源。

压力实验只针对自己的测试实例，设定时间与总资源上限；不对整台共享服务器施压。保持默认 seccomp，不把禁用过滤当作常规解决方案。

## 5. 验收

输出一张信任边界图：哪些输入来自客户机、哪些组件在宿主高权限侧、哪些路径承载敏感管理操作。至少提供一次 CPU 限制和一次内存边界验证的证据。
