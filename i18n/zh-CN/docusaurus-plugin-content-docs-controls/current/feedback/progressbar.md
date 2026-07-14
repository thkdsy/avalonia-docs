---
id: progressbar
title: ProgressBar
description: 一个以填充比例显示数值的水平或垂直条形控件，并可选支持文本说明和未知进度的不确定模式。
doc-type: reference
---

`ProgressBar` 将一个值显示为按比例填充的条形，并可选择显示说明文本。你可以用它来表示文件下载、安装过程或数据处理任务等长时间操作的完成状态。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
|----------------------|-------------------------------------------------------------------------------------------------|
| `Minimum` | 范围的最小值。默认值为 `0`。 |
| `Maximum` | 范围的最大值。默认值为 `100`。 |
| `Value` | 当前范围内的值。 |
| `IsIndeterminate` | 为 `true` 时，条形显示为动画指示器，而不是按比例填充。 |
| `Orientation` | 设置条形方向。可选 `Horizontal`（默认）或 `Vertical`。 |
| `Foreground` | 用于绘制填充部分的画刷。 |
| `ShowProgressText` | 为 `true` 时，进度条会叠加显示当前进度的文本说明。 |
| `ProgressTextFormat` | 控制进度文本渲染方式的格式字符串。详见下文。 |

## 示例

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Margin="20">
  <ProgressBar  Margin="0 10" Height="20"
                Minimum="0" Maximum="100" Value="14"
                ShowProgressText="True"/>
  <ProgressBar  Margin="0 10" Height="20"
                Minimum="0" Maximum="100" Value="92"
                Foreground="Red"
                ShowProgressText="True"/>
</StackPanel>
```

</XamlPreview>

## 不确定模式

当总工作量未知时，请将 `IsIndeterminate` 设置为 `True`。此时条形不会显示填充比例，而是显示循环动画，用于表明操作正在进行中，但无法给出具体完成百分比。

```xml
<ProgressBar IsIndeterminate="True" Height="20" />
```

这对于连接远程服务器、等待外部进程完成，或加载大小未知的数据等操作非常有用。

若要切回确定模式，请将 `IsIndeterminate` 设置为 `False`，并随着操作推进更新 `Value`：

```xml
<ProgressBar IsIndeterminate="{Binding IsLoading}"
             Value="{Binding Progress}"
             Minimum="0" Maximum="100"
             Height="20" />
```

## 使用 `ProgressTextFormat` 自定义进度文本

默认情况下，`ShowProgressText` 会显示根据以下属性计算出的完成百分比：
[`Value`](/api/avalonia/controls/primitives/rangebase#value-property),
[`Minimum`](/api/avalonia/controls/primitives/rangebase#minimum-property), and
[`Maximum`](/api/avalonia/controls/primitives/rangebase#maximum-property)。你可以通过将 `ProgressTextFormat` 设置为格式字符串来自定义显示文本。该字符串会传递给 [`string.Format`](https://docs.microsoft.com/en-us/dotnet/api/system.string.format#system-string-format(system-string-system-object()))，可使用以下格式项：

| 索引 | 说明 |
|-------|----------------------------------------------------------------------------------------------------------------|
| `0` | 当前 `Value`。 |
| `1` | 以 0 到 100 百分比表示的值（例如，`Minimum = 0`、`Maximum = 50`、`Value = 25` 时会得到 `50`）。 |
| `2` | `Minimum` 值。 |
| `3` | `Maximum` 值。 |

| Min | Max | Value | `ProgressTextFormat`                | Output                       |
|-----|-----|-------|-------------------------------------|------------------------------|
| 0   | 20  | 17    | `{}{0}/{3} Tasks Complete ({1:0}%)` | `17/20 Tasks Complete (85%)` |

因为在这个示例中 `{0}` 会出现在字符串开头，所以你必须在前面加上 `{}` 进行转义。

## 垂直方向

你可以通过设置 `Orientation` 属性让进度条以垂直方式显示：

```xml
<ProgressBar Orientation="Vertical" Height="200" Width="20"
             Minimum="0" Maximum="100" Value="65"
             ShowProgressText="True" />
```

## 绑定到视图模型

将 `Value` 绑定到视图模型中的属性，以跟踪异步操作的进度：

```xml
<ProgressBar Minimum="0" Maximum="100"
             Value="{Binding DownloadProgress}"
             ShowProgressText="True"
             ProgressTextFormat="{}{1:0}%" />
```

```csharp
[ObservableProperty]
private double _downloadProgress;

public async Task DownloadFileAsync()
{
    for (int i = 0; i <= 100; i += 10)
    {
        await Task.Delay(500);
        DownloadProgress = i;
    }
}
```

## 样式设置

你可以通过主题资源或控件模板部件来自定义 `ProgressBar` 的样式。该控件暴露了以下关键模板部件：

| 部件名称 | 说明 |
|------------------------|-------------------------------------------------------|
| `PART_Indicator` | 表示已填充区域的 `Border` 元素。 |
| `PART_ProgressBarText` | 显示进度文本说明的 `TextBlock`。 |

若要更改轨道背景或进度指示器颜色，可以重写相关主题资源，或直接设置属性：

```xml
<ProgressBar Height="20" Value="60" Maximum="100"
             Foreground="Green"
             Background="LightGray" />
```

如果需要更高级的自定义，你可以提供完整的 `ControlTheme`：

```xml
<ProgressBar Height="20" Value="50" Maximum="100">
  <ProgressBar.Styles>
    <Style Selector="ProgressBar /template/ Border#PART_Indicator">
      <Setter Property="CornerRadius" Value="4" />
    </Style>
  </ProgressBar.Styles>
</ProgressBar>
```

## 另请参阅

- [Slider](/controls/input/selectors/slider)
- [ProgressBar API reference](/api/avalonia/controls/progressbar)
- [`ProgressBar.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ProgressBar.cs)
