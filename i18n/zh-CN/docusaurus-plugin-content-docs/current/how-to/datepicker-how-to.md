---
id: datepicker-how-to
title: "如何：使用日期和时间选择器"
description: 学习如何在 Avalonia 选择器中绑定日期、设置显示格式、验证输入以及配置日期范围。
doc-type: how-to
---

本指南介绍 `DatePicker`、`TimePicker`、`CalendarDatePicker` 和 `Calendar` 的常见使用模式，包括日期绑定、格式化、验证以及日期范围控制。

## DatePicker 基础用法

`DatePicker` 使用滚轮式选择控件来选择日、月和年：

```xml
<DatePicker />
```

### 设置初始日期

日期属性必须在代码中设置，而不能直接写成 XAML 属性字符串，因为目前没有内置的字符串到 `DateTimeOffset` 的转换器：

```csharp
myDatePicker.SelectedDate = new DateTimeOffset(new DateTime(2025, 6, 15));
```

或者将其绑定到视图模型属性：

```csharp
[ObservableProperty]
private DateTimeOffset? _birthDate;
```

```xml
<DatePicker SelectedDate="{Binding BirthDate}" />
```

## 自定义日期格式

你可以分别控制日期各部分的显示方式：

```xml
<!-- 显示缩写星期名 -->
<DatePicker DayFormat="ddd dd" />

<!-- 显示完整月份名 -->
<DatePicker MonthFormat="MMMM" />

<!-- 四位年份 -->
<DatePicker YearFormat="yyyy" />
```

常见格式字符串如下：

| Format | Output example |
|---|---|
| `d` | 5 |
| `dd` | 05 |
| `ddd` | Mon |
| `dddd` | Monday |
| `M` | 6 |
| `MM` | 06 |
| `MMM` | Jun |
| `MMMM` | June |
| `yy` | 25 |
| `yyyy` | 2025 |

## 隐藏日期部分

只显示你需要的字段：

```xml
<!-- 仅显示月份和年份（不显示日期） -->
<DatePicker DayVisible="False" />

<!-- 仅显示年份 -->
<DatePicker DayVisible="False" MonthVisible="False" />
```

## TimePicker

`TimePicker` 提供小时和分钟滚轮选择：

```xml
<TimePicker />
```

### 12 小时制与 24 小时制

```xml
<!-- 12 小时制，带 AM/PM -->
<TimePicker ClockIdentifier="12HourClock" />

<!-- 24 小时制 -->
<TimePicker ClockIdentifier="24HourClock" />
```

### 分钟步进

```xml
<!-- 每 15 分钟一个步进 -->
<TimePicker MinuteIncrement="15" />
```

### 绑定所选时间

```csharp
[ObservableProperty]
private TimeSpan? _alarmTime;
```

```xml
<TimePicker SelectedTime="{Binding AlarmTime}" />
```

## CalendarDatePicker

`CalendarDatePicker` 会显示一个文本框，点击后可展开完整日历下拉面板：

```xml
<CalendarDatePicker PlaceholderText="Select a date"
                    SelectedDate="{Binding EventDate}" />
```

### 显示格式

```xml
<CalendarDatePicker DisplayFormat="yyyy-MM-dd"
                    SelectedDate="{Binding EventDate}" />
```

### 不可选日期

你可以禁用某些特定日期，使其不能被选择：

```csharp
calendarDatePicker.BlackoutDates.Add(
    new CalendarDateRange(DateTime.Today, DateTime.Today.AddDays(3)));
```

## Calendar 控件

`Calendar` 会以内嵌方式显示完整月份视图，用于日期选择：

```xml
<Calendar SelectedDate="{Binding SelectedDate}"
          SelectionMode="SingleDate" />
```

### 选择模式

```xml
<!-- 单个日期 -->
<Calendar SelectionMode="SingleDate" />

<!-- 一段连续日期 -->
<Calendar SelectionMode="SingleRange" />

<!-- 多个独立日期 -->
<Calendar SelectionMode="MultipleRange" />

<!-- 不允许选择（仅显示） -->
<Calendar SelectionMode="None" />
```

### 显示模式

```xml
<!-- 显示月份视图（默认） -->
<Calendar DisplayMode="Month" />

<!-- 从年份选择开始 -->
<Calendar DisplayMode="Year" />

<!-- 从十年选择开始 -->
<Calendar DisplayMode="Decade" />
```

### 日期范围限制

限制可浏览的日期范围：

```xml
<Calendar DisplayDateStart="2025-01-01"
          DisplayDateEnd="2025-12-31" />
```

### 在代码中设置不可选日期

```csharp
// 禁用周末
for (var date = startDate; date <= endDate; date = date.AddDays(1))
{
    if (date.DayOfWeek is DayOfWeek.Saturday or DayOfWeek.Sunday)
        calendar.BlackoutDates.Add(new CalendarDateRange(date));
}
```

## 日期验证

验证用户所选日期是否在允许范围内：

```csharp
public partial class BookingViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [CustomValidation(typeof(BookingViewModel), nameof(ValidateFutureDate))]
    private DateTimeOffset? _departureDate;

    public static ValidationResult? ValidateFutureDate(DateTimeOffset? date, ValidationContext context)
    {
        if (date.HasValue && date.Value.Date <= DateTimeOffset.Now.Date)
            return new ValidationResult("Departure date must be in the future.");
        return ValidationResult.Success;
    }
}
```

## 在显示中格式化日期

在 `TextBlock` 中显示格式化后的已选日期：

```xml
<StackPanel Spacing="8">
    <DatePicker SelectedDate="{Binding SelectedDate}" />
    <TextBlock Text="{Binding SelectedDate, StringFormat='Selected: {0:d}'}" />
</StackPanel>
```

## 组合日期与时间

可以把两个选择器一起使用，以获得完整的日期时间输入：

```xml
<StackPanel Orientation="Horizontal" Spacing="12">
    <DatePicker SelectedDate="{Binding EventDate}" />
    <TimePicker SelectedTime="{Binding EventTime}" ClockIdentifier="24HourClock" />
</StackPanel>
```

然后在视图模型中把它们组合起来：

```csharp
public DateTime? CombinedDateTime
{
    get
    {
        if (EventDate is null) return null;
        var date = EventDate.Value.Date;
        return EventTime.HasValue
            ? date.Add(EventTime.Value)
            : date;
    }
}
```

## 关键属性参考

### DatePicker

| 属性 | 类型 | 说明 |
|---|---|---|
| `SelectedDate` | `DateTimeOffset?` | 当前选中的日期。 |
| `DayVisible` | `bool` | 是否显示日期滚轮。 |
| `MonthVisible` | `bool` | 是否显示月份滚轮。 |
| `YearVisible` | `bool` | 是否显示年份滚轮。 |
| `DayFormat` | `string` | 日期部分的显示格式。 |
| `MonthFormat` | `string` | 月份部分的显示格式。 |
| `YearFormat` | `string` | 年份部分的显示格式。 |

### TimePicker

| 属性 | 类型 | 说明 |
|---|---|---|
| `SelectedTime` | `TimeSpan?` | 当前选中的时间。 |
| `ClockIdentifier` | `string` | `"12HourClock"` 或 `"24HourClock"`。 |
| `MinuteIncrement` | `int` | 分钟滚轮的步进值。 |

## 另请参阅

- [DatePicker Control Reference](/controls/input/date-and-time/datepicker)：属性与接口说明。
- [TimePicker Control Reference](/controls/input/date-and-time/timepicker)：时间选择控件说明。
- [Calendar Control Reference](/controls/input/date-and-time/calendar)：完整日历控件说明。
- [Data Validation](/docs/data-binding/binding-validation)：绑定值验证方式。
