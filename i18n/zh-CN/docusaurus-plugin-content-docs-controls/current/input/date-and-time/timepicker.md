---
id: timepicker
title: TimePicker
description: 一个允许用户通过小时、分钟以及可选秒数的微调控件来选择时间值的控件。
doc-type: reference
---

`TimePicker` 提供两个到四个微调控件，允许用户选择一个时间值。它支持 24 小时制和 12 小时制，并可选择是否启用秒数选择。点击控件时会显示这些微调控件。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `SelectedTime` | `TimeSpan?` | 选中的时间值。未选择时间时为 `null`。 |
| `ClockIdentifier` | `string` | 设置时钟格式。可使用 `12HourClock` 或 `24HourClock`。12 小时制会额外显示 AM/PM 微调器。 |
| `UseSeconds` | `bool` | 为 `true` 时显示额外的秒数微调器。默认值为 `false`。 |
| `MinuteIncrement` | `int` | 定义分钟的可选步进值。默认值为 `1`。 |
| `SecondIncrement` | `int` | 定义秒数的可选步进值。默认值为 `1`。 |

## 时钟格式

默认情况下，`TimePicker` 使用带有 AM/PM 微调器的 12 小时制。你可以通过将 `ClockIdentifier` 设置为 `24HourClock` 切换到 24 小时制：

```xml
<!-- 带 AM/PM 微调器的 12 小时制（默认） -->
<TimePicker ClockIdentifier="12HourClock" />

<!-- 不带 AM/PM 微调器的 24 小时制 -->
<TimePicker ClockIdentifier="24HourClock" />
```

## 示例

此示例展示了如何创建一个使用 24 小时制、并以 20 分钟为时间间隔的时间选择器：

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Margin="20"
            Spacing="4">
  <Label Content="Please choose your time:"/>
  <TimePicker ClockIdentifier="24HourClock"
              MinuteIncrement="20"/>
</StackPanel>
```
</XamlPreview>

## 初始化时间

你可以在 XAML 中将时间值作为属性直接设置。请使用 `Hh:Mm` 形式的字符串，其中 `Hh` 表示小时（0 到 23），`Mm` 表示分钟（0 到 59）：

```xml
<TimePicker SelectedTime="09:15"/>
```

如果你需要在 code-behind 中编写代码，可以像这样初始化时间：

```csharp
TimePicker timePicker = new TimePicker
{
    SelectedTime = new TimeSpan(9, 15, 0) // 秒数会被忽略。
};
```

你可以通过将 `SelectedTime` 重置为 `null` 来清空显示内容。

## 限制时间范围

你可以通过调整 `MinuteIncrement` 和 `SecondIncrement` 来限制可选时间。例如，仅允许以 15 分钟为间隔选择：

```xml
<TimePicker MinuteIncrement="15" />
```

若要启用秒数并以 30 秒为间隔：

```xml
<TimePicker UseSeconds="True" SecondIncrement="30" />
```

## 视图模型绑定

将 `SelectedTime` 绑定到视图模型中的 `TimeSpan?` 属性：

```xml
<TimePicker SelectedTime="{Binding AppointmentTime}"
            ClockIdentifier="12HourClock" />
```

```csharp
[ObservableProperty]
private TimeSpan? _appointmentTime = new TimeSpan(14, 30, 0);
```

你可以通过订阅 `SelectedTimeChanged` 事件，或在视图模型中观察属性变化来响应时间变更。

## 另请参阅

- [DatePicker](/controls/input/date-and-time/datepicker)
- [CalendarDatePicker](/controls/input/date-and-time/calendardatepicker)
- [TimePicker API reference](/api/avalonia/controls/timepicker)
- [`TimePicker.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/DateTimePickers/TimePicker.cs)
