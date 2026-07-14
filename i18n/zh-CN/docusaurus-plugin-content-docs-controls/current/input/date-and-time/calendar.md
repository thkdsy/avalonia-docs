---
id: calendar
title: Calendar
---

import CalendarBasicUsageScreenshot from '/img/controls/calendar/calendar3.gif';
import CalendarSingleSelectionScreenshot from '/img/controls/calendar/calendar.gif';
import CalendarMultipleSelectionScreenshot from '/img/controls/calendar/calendar2.gif';
import CalendarCustomRangeScreenshot from '/img/controls/calendar/calendar4.gif';

Calendar 是一个供用户选择日期或日期范围的控件。

<Image light={CalendarBasicUsageScreenshot} alt="An animation of a calendar switching between year, month and day views." position="center" maxWidth={400} cornerRadius="true"/>

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="251">属性</th><th>说明</th></tr></thead><tbody><tr><td><code>SelectionMode</code></td><td>指示允许的选择类型。可选值包括：单个日期、单个范围、多个范围以及不允许选择。</td></tr><tr><td><code>DisplayMode</code></td><td>定义日历在层级钻取中的初始显示级别。可选值包括：十年、年份和月份（默认）。</td></tr><tr><td><code>SelectedDate</code></td><td>当前选中的日期。</td></tr><tr><td><code>SelectedDates</code></td><td>已选日期的集合，包含单个范围和多个范围中的所有日期。</td></tr><tr><td><code>DisplayDate</code></td><td>控件首次显示时要展示的日期。</td></tr><tr><td><code>DisplayDateStart</code></td><td>允许显示的最早日期。</td></tr><tr><td><code>DisplayDateEnd</code></td><td>允许显示的最晚日期。</td></tr><tr><td><code>BlackoutDates</code></td><td>显示为不可用且无法被选中的日期集合。</td></tr><tr><td><code>AllowTapRangeSelection</code></td><td>当为 <code>true</code>（默认值）时，允许通过先点按起始日期再点按结束日期的方式选择日期范围。适用于 <code>SingleRange</code> 和 <code>MultipleRange</code> 选择模式。</td></tr></tbody></table>

## 示例

这是一个允许选择单个日期的基础日历示例。日历当前选中的日期会显示在下方的文本块中。

```xml
<StackPanel Margin="20">
  <Calendar SelectionMode="SingleDate"/>
  <TextBlock Margin="20" 
             Text="{Binding #calendar.SelectedDate}"/>
</StackPanel>
```

<Image light={CalendarSingleSelectionScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

此示例允许选择多个日期范围：

```xml
  <StackPanel Margin="20">
    <Calendar SelectionMode="MultipleRange"/>
  </StackPanel>
```

<Image light={CalendarMultipleSelectionScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

若要选择一个范围，请先点按或单击起始日期，再点按或单击结束日期。这种点按选择行为由 `AllowTapRangeSelection` 属性控制，且默认启用。你也可以按住 Shift 键并单击结束日期来扩展范围。按住 Ctrl 键并单击其他日期，则可以追加更多日期和范围。

此示例设置了自定义起止日期，并让其中某些日期不可用。这里使用了窗口的 C# 代码后置。

```xml
<UserControl xmlns="https://github.org/avaloniaui">
<StackPanel Margin="20">
  <Calendar x:Name="calendar" SelectionMode="SingleDate"/>
</StackPanel>
</UserControl>
```


```csharp title='C#'
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        var today = DateTime.Today;
        calendar.DisplayDateStart = today.AddDays(-25);
        calendar.DisplayDateEnd = today.AddDays(25);
        calendar.BlackoutDates.Add(
            new CalendarDateRange( today.AddDays(5), today.AddDays(10)));
    } 
}
```


<Image light={CalendarCustomRangeScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [Calendar API 参考](/api/avalonia/controls/calendar)
- [`Calendar.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Calendar/Calendar.cs)
