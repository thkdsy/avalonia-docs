---
id: calendardatepicker
title: CalendarDatePicker
description: 一个带有文本框的下拉日历控件，允许用户选择或输入日期。
doc-type: reference
---

import CalendarDatePickerScreenshot from '/img/gitbook-import/assets/calendardatepicker.gif';

`CalendarDatePicker` 结合了一个文本框和一个下拉按钮，按钮会展开一个完整日历。点击按钮时，日历会打开，便于你通过可视方式选择日期。再次点击按钮（或直接选择某个日期）会关闭日历，并将所选日期填入文本框。

你也可以直接在文本框中输入日期。该控件接受多种日期格式，并会在没有选中日期时将其标准化为占位文本所显示的格式。

:::info
有关该控件中日历部分的详细说明，请参阅 [Calendar](/controls/input/date-and-time/calendar) 文档。
:::

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `SelectedDate` | `DateTime?` | 当前选中的日期；如果未选择日期，则为 `null`。 |
| `DisplayDate` | `DateTime` | 日历打开时显示的月份。 |
| `DisplayDateStart` | `DateTime?` | 可选择的最早日期。 |
| `DisplayDateEnd` | `DateTime?` | 可选择的最晚日期。 |
| `PlaceholderText` | `string` | 未选择日期时显示的占位文本。 |
| `PlaceholderForeground` | `IBrush` | 用于绘制占位文本的画刷。 |
| `IsTodayHighlighted` | `bool` | 是否高亮显示今天的日期。默认值为 `true`。 |
| `SelectedDateFormat` | `CalendarDatePickerFormat` | 显示格式：`Short` 或 `Long`。 |
| `CustomDateFormatString` | `string` | 使用自定义格式时的日期格式字符串。 |
| `IsDropDownOpen` | `bool` | 日历下拉框当前是否已打开。 |

## 绑定到视图模型

```xml title="XAML"
<CalendarDatePicker SelectedDate="{Binding BirthDate}"
                    PlaceholderText="Select date of birth"
                    DisplayDateEnd="{Binding Today}" />
```

```csharp title="C#"
[ObservableProperty]
private DateTimeOffset? _birthDate;

public DateTimeOffset Today { get; } = DateTimeOffset.Now;
```

## 日期范围限制

你可以通过 `DisplayDateStart` 和 `DisplayDateEnd` 来限制可选日期范围：

```xml title="XAML"
<CalendarDatePicker SelectedDate="{Binding CheckInDate}"
                    DisplayDateStart="2024-01-01"
                    DisplayDateEnd="2025-12-31"
                    PlaceholderText="Check-in date" />
```

## 实用说明

- **输入日期**：当用户输入的日期超出 `DisplayDateStart`/`DisplayDateEnd` 范围时，控件会拒绝该值并清空文本框。
- **空值处理**：将 `SelectedDate` 绑定到可空的 `DateTimeOffset?` 属性，这样控件才能表示“未选择”。
- **格式自定义**：将 `SelectedDateFormat` 设置为 `CalendarDatePickerFormat.Custom`，并提供 `CustomDateFormatString`（例如 `"yyyy-MM-dd"`），以控制所选日期在文本框中的显示方式。
- **键盘支持**：用户可以使用 `Alt+Down` 打开下拉框，并用 `Escape` 关闭它。

## 示例

此示例展示了一个基础的单日期选择日历，点击按钮即可打开：

<XamlPreview>

```xml title="XAML"
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <StackPanel Margin="20">
    <CalendarDatePicker />
  </StackPanel>
</UserControl>
```

</XamlPreview>

## 另请参阅

- [Calendar](/controls/input/date-and-time/calendar)
- [DatePicker](/controls/input/date-and-time/datepicker)
- [TimePicker](/controls/input/date-and-time/timepicker)
- [CalendarDatePicker API reference](/api/avalonia/controls/calendardatepicker)
- [`CalendarDatePicker.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/CalendarDatePicker/CalendarDatePicker.cs)
