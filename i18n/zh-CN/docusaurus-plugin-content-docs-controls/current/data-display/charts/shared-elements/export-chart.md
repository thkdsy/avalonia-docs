---
id: export-chart
title: 图表导出
description: 使用异步 API 将图表控件导出为 PNG 或 JPEG 文件、PNG 流或交互式保存对话框。
doc-type: reference
tags:
  - avalonia pro
---

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

图表控件继承自 `ChartBase` 的导出 API。使用 `ExportAsync` 将当前图表视图渲染为图像文件、PNG 流或保存对话框。

## 使用时机

- **报告**：为生成的报告保存图表图像。
- **用户导出**：让用户从应用程序中保存当前图表视图。
- **快照**：为需要位图输出的工作流渲染图表图像。

## 代码示例

### 导出到文件

```csharp
var result = await chart.ExportAsync("sales-chart.png", width: 1200, height: 800, dpi: 144);

if (!result.Succeeded)
{
    // 使用 result.Canceled 或 result.Exception 处理结果。
}
```

### 导出到流

```csharp
await using var stream = System.IO.File.Create("sales-chart.png");
var result = await chart.ExportAsync(stream, width: 1200, height: 800);
```

## 方法

| 方法 | 描述 | 结果 |
| :--- | :--- | :--- |
| `ExportAsync(string path, int? width = null, int? height = null, double dpi = 96, CancellationToken cancellationToken = default)` | 导出到文件路径。如果路径没有扩展名，则追加 `.png`。文件导出支持 PNG 和 JPEG。 | `ChartExportResult` |
| `ExportAsync(Stream stream, int? width = null, int? height = null, double dpi = 96, CancellationToken cancellationToken = default)` | 将 PNG 图像导出到流。 | `ChartExportResult` |
| `ExportAsync(CancellationToken cancellationToken = default)` | 打开文件保存选择器并导出到所选文件。 | `ChartExportResult` |

## 结果和事件

| 成员 | 描述 |
| :--- | :--- |
| `ChartExportResult.Succeeded` | 导出成功完成时为 `true`。 |
| `ChartExportResult.Canceled` | 导出被取消时为 `true`。 |
| `ChartExportResult.Target` | 文件路径或目标描述，例如 `stream`。 |
| `ChartExportResult.Exception` | 导致导出失败的异常（如有）。 |
| `ExportCompleted` | 导出成功后引发。事件数据包含 `Result` 和 `Target`。 |
| `ExportFailed` | 导出失败后引发。事件数据包含 `Target` 和 `Exception`。 |

## 说明

- `width` 和 `height` 默认为图表边界。它们必须解析为正数。
- 取消操作会返回已取消的 `ChartExportResult`，且不会引发 `ExportFailed`。
- 无效路径、无效尺寸和流错误会返回失败的 `ChartExportResult`，并引发 `ExportFailed`。

## 另请参阅

- [交互](/controls/data-display/charts/shared-elements/interactions-chart)
- [图例](/controls/data-display/charts/shared-elements/legend-chart)
