# 专题：日志、指标、性能与容量

[统一课程](../README.md) · [面试题与答案](../expert-assessment/by-module/README.md)

这是按主题组织的阅读与实验资料；课程模块及考核以总目录为准。


前置：至少能稳定运行一台 microVM。目标：用可复现实验回答“为什么慢”和“还能接多少任务”。

## 1. 先学基础

**延迟、吞吐与利用率。**单任务完成时间、每秒完成任务数和资源占用分别描述不同维度。高利用率可能增加排队和尾延迟，不能单独作为性能好的证据。

**分位数。**P95 表示观测样本中约 95% 的延迟不超过该值。只有少数样本时，P99 很不稳定；应同时报告样本数、错误数和分布，不从三次启动推断生产尾延迟。

**分段测量。**总就绪时间可以拆为排队、资源准备、进程/API 启动、客户机启动或恢复、应用初始化与健康检查。部分工作可并行，要以时间线识别关键路径，而不是随意相加重叠区间。

**Little 定律。**稳定系统中平均在系统任务数 `L = λ × W`。当 `W` 只计算执行时长时，得到平均执行并发；包括排队时长则得到排队与执行总数。它不直接给出峰值或尾延迟所需容量。

## 2. 资料阅读

- [Logger](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/logger.md) 与 [Metrics](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/metrics.md)：分别配置日志和 JSON 指标，区分客户机输出与 VMM 日志。
- [启动时间测试](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/tests/integration_tests/performance/test_boottime.py)：检查测试实际计时边界。
- [内存开销测试](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/tests/integration_tests/performance/test_memory_overhead.py)：理解不同内存指标的口径。
- [网络性能](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/network-performance.md) 与 [快照性能测试](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/tests/integration_tests/performance/test_snapshot.py)。
- 进阶：[Tracing](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/docs/tracing.md) 与 [测试运行说明](https://github.com/firecracker-microvm/firecracker/blob/30471852666564d980f330d0575115eda7d5ce8e/tests/README.md)。

## 3. 建立三层观测

| 层 | 至少观察什么 | 解决的问题 |
| --- | --- | --- |
| 宿主机 | CPU、内存、cgroup 事件、磁盘与网络压力 | 整机是否饱和或超限 |
| Firecracker | API 错误、设备指标、VMM 日志 | VM 配置和设备路径是否异常 |
| 客户机与应用 | 启动标记、健康检查、任务退出码 | 业务是否真正可用和完成 |

每轮关联实例 ID、任务 ID、镜像版本、资源配置与时间戳。计算持续时间优先使用同一测量端的单调时钟，避免宿主与客户机时钟差影响结果。指标字段及计数口径以所用版本定义为准；不要假定所有字段都是进程启动以来的累计值。

## 4. 实验：一个小型基准矩阵

1. 固定版本、镜像、CPU/内存配置和健康检查。确认实验期间有足够宿主资源。
2. 分别执行冷启动和快照恢复，每种先做至少 30 次探索性样本；这是发现问题的起点，不足以宣称稳定 P99。
3. 区分“完整启动客户机”与“宿主文件缓存冷/热”。没有真正控制文件缓存的实验就明确标注；不要在共享服务器上全局清缓存。
4. 资源允许时比较 1、2、4 个并发实例，并设置停止阈值。记录总就绪时间、首次请求延迟、失败数、CPU、内存和 I/O。
5. 找到最慢阶段，只改变一个变量重测；给出优化前后证据与代价。

报告模板：环境 → 假设 → 测量边界 → 样本与失败 → 分布 → 瓶颈证据 → 修改 → 结果 → 未验证范围。

## 5. 容量估算练习

以下都是教学假设，不能当作 Firecracker 固定开销：一台 64 GiB、16 核机器，预留 8 GiB；每任务客户机内存与宿主侧预算合计 320 MiB；每个运行任务平均消耗 0.1 核；CPU 目标利用率不超过 75%。

- 内存上限估算：`floor(56 × 1024 / 320) = 179` 个。
- CPU 上限估算：`floor(16 × 0.75 / 0.1) = 120` 个。
- 初步约束取较小值 120，随后还要校验磁盘、网络、启动尖峰和应用尾延迟。

平均 CPU 不是每任务的保证。若同时进入 CPU 密集阶段，需要更保守的准入或明确的超售策略。磁盘满也可能先于 CPU/内存成为限制。
