---
id: metrics-tool
title: Metrics 工具
description: 在 Avalonia Developer Tools 中使用计数器、直方图和其他 .NET 指标仪表监控应用健康状况，并创建自定义指标源。
doc-type: reference
---

Metrics 是随时间上报的数值测量，通常用于监控应用健康状态并生成告警。

Meter provider 从 .NET 6 开始引入。越来越多的库以及 .NET 自身的部分组件（例如 HttpClient）已经开始支持它，从而提供实用的诊断数据。

指标仪表主要有四种类型：
- Counter - 表示一个只能增长的单值测量。通常用于表示“总量”值，例如异常总数。
- UpDownCounter - 与 Counter 类似，但允许负向增量。例如内存工作集。
- Histogram - 测量值分布。例如帧渲染耗时或 HTTP 请求耗时。对于直方图，`Developer Tools` 还会显示有用的 P50（中位数）、P90 和 P95 百分位。
- Gauge - 不包含历史数据的测量，只显示最新值。**注意**：`Developer Tools` 当前尚不支持。

![Histogram](/img/tools/dev-tools/metrics-histogram.png)

## 禁用/启用默认源

默认情况下，`Developer Tools` 只会接收 `Avalonia` 和 `System.Runtime` 的 meter provider。

你可以点击 **+** 按钮来禁用它们，或启用其他 provider。
所有仪表都会按其 meter provider 分组，通常这个名称就是定义它的命名空间。

请注意，这个列表是动态的。只有当 provider 至少推送过一次测量值后，新仪表才会被加入列表。例如，HttpClient（`System.Http` 命名空间）的仪表只有在首次请求发生后才会显示出来。

![Meters Filter](/img/tools/dev-tools/meters-filter.png)

:::note

请注意，`Avalonia` meters 从框架版本 11.3.0 起才可用，而 `System.Runtime` 则仅在 .NET 9 起可用。
如果你的应用不满足这两个条件中的任意一个，那么默认情况下你会看到空的 meters 列表。

:::

## 编写自定义指标源

你可以参考 .NET 的 [Creating metrics](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/metrics-instrumentation) 文档，了解如何编写自定义指标源，然后在 `Developer Tools` 或官方 `dotnet-counters` 控制台工具中查看这些指标。

最简单的示例大致如下：

```csharp
static Meter s_meter = new Meter("SimpleToDoList");
static UpDownCounter<int> s_tasksCount = s_meter.CreateUpDownCounter<int>("tasks.count");
static Counter<int> s_tasksResolved = s_meter.CreateCounter<int>("tasks.resolved.total");

private void OnTaskAdded() => s_tasksCount.Add(1);
private void OnTaskRemoved() => s_tasksCount.Add(-1);
private void OnTaskResolved() => s_tasksResolved.Add(1);
```

当代码至少推送过一次测量值后，这个 `SimpleToDoList` 就会出现在 **+** 按钮的浮出菜单中，并可显示在该工具里。

## 另请参阅

- [Profiler 工具](/tools/developer-tools/profiler-tool)
- [Developer tools 安装](/tools/developer-tools/installation)
