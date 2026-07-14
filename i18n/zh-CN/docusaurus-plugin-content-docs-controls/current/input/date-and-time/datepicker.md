---
id: datepicker
title: DatePicker
description: 一个基于微调器的控件，允许用户通过选择年、月、日来选取日期。
doc-type: reference
---

import DatePickerScreenshot from '/img/controls/datepicker/datepicker.gif';

`DatePicker` 控件提供三个微调列，让用户可以选择一个日期值。点击控件时，这些微调器会显示出来。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
|---|---|
| `SelectedDate` | 以 `DateTimeOffset?` 表示的已选日期（未选择时为 null）。 |
| `DayVisible` | 设置日期列是否可见。 |
| `MonthVisible` | 设置月份列是否可见。 |
| `YearVisible` | 设置年份列是否可见。 |
| `DayFormat` | 日期部分的格式字符串。 |
| `MonthFormat` | 月份部分的格式字符串。 |
| `YearFormat` | 年份部分的格式字符串。 |
| `MinYear` | 可选的最小年份。 |
| `MaxYear` | 可选的最大年份。 |

## 示例

此示例使用 `DayFormat` 属性同时显示星期名称和日期数字：

```xml
<StackPanel Margin="20">
  <DatePicker DayFormat="ddd dd"/>
</StackPanel>
```

<Image light={DatePickerScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 隐藏日期部分

你可以通过将可见性属性设置为 `False`，仅显示所需的日期组成部分：

```xml
<!-- 仅显示月份和年份 -->
<DatePicker DayVisible="False" />

<!-- 仅显示年份 -->
<DatePicker DayVisible="False" MonthVisible="False" />
```

## 限制日期范围

使用 `MinYear` 和 `MaxYear` 可以限制用户可选择的年份范围。当你需要将选择限制在一个已知有效范围内时，这会很有用，例如出生日期或到期日期。

```xml
<DatePicker MinYear="2000/01/01" MaxYear="2030/12/31" />
```

你也可以在 code-behind 中设置这些值：

```csharp
datePicker.MinYear = new DateTimeOffset(new DateTime(2000, 1, 1));
datePicker.MaxYear = new DateTimeOffset(new DateTime(2030, 12, 31));
```

## 自定义显示格式

选择器中的每一列都支持标准的 .NET 日期格式字符串。你可以组合这些格式，以精确控制用户看到的内容：

```xml
<!-- 完整月份名称、缩写星期名称、四位年份 -->
<DatePicker MonthFormat="MMMM" DayFormat="ddd dd" YearFormat="yyyy" />

<!-- 数字月份、仅日期数字、两位年份 -->
<DatePicker MonthFormat="MM" DayFormat="dd" YearFormat="yy" />
```

## 初始化日期

该控件的日期属性不能在 AXAML 中通过字符串属性直接设置，因为内置并不支持从字符串到 `DateTimeOffset` 的转换。

你可以在 code-behind 中设置该值：

```csharp
datePicker.SelectedDate = new DateTimeOffset(new DateTime(1950, 1, 1));
```

## 绑定到视图模型

在大多数应用中，你会将 `SelectedDate` 绑定到视图模型中的某个属性。该属性应为可空的 `DateTimeOffset`，以便表示“未选择”状态。

```csharp
public class MyViewModel : ObservableObject
{
    [ObservableProperty]
    private DateTimeOffset? _selectedDate;
}
```

```xml
<DatePicker SelectedDate="{Binding SelectedDate}" />
```

如果你需要在用户更改日期时作出响应，可以订阅 `SelectedDateChanged` 事件，或在视图模型中使用属性变更回调：

```csharp
public partial class MyViewModel : ObservableObject
{
    [ObservableProperty]
    private DateTimeOffset? _selectedDate;

    partial void OnSelectedDateChanged(DateTimeOffset? value)
    {
        // 在这里响应新的日期值。
    }
}
```

## 另请参阅

- [Calendar](/controls/input/date-and-time/calendar)
- [CalendarDatePicker](/controls/input/date-and-time/calendardatepicker)
- [TimePicker](/controls/input/date-and-time/timepicker)
- [DatePicker API reference](/api/avalonia/controls/datepicker)
- [`DatePicker.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/DateTimePickers/DatePicker.cs)
