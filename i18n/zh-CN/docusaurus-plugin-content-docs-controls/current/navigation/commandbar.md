---
title: CommandBar
description: '`CommandBar` 是一种用于显示一排命令按钮的工具栏。'
doc-type: reference
---

import CommandBarLabelBottomScreenshot from '/img/controls/commandbar/commandbar-label-bottom.png';
import CommandBarLabelRightScreenshot from '/img/controls/commandbar/commandbar-label-right.png';
import CommandBarLabelCollapsedScreenshot from '/img/controls/commandbar/commandbar-label-collapsed.png';
import CommandBarSecondaryCommandsScreenshot from '/img/controls/commandbar/commandbar-secondary-commands.png';
import CommandBarContentScreenshot from '/img/controls/commandbar/commandbar-content.png';
import CommandBarToggleButtonScreenshot from '/img/controls/commandbar/commandbar-toggle-button.png';

`CommandBar` 是一种工具栏，用于显示一排主命令按钮，并可选提供一个用于显示次级命令的溢出菜单。主命令始终显示在工具栏中。次级命令则会出现在由溢出按钮（`...`）打开的弹出菜单中。当 `IsDynamicOverflowEnabled` 为 `true` 时，那些无法适应可用宽度的主命令会自动移动到溢出区域。

`CommandBar` 中的项目实现了 `ICommandBarElement` 接口。内置实现包括 `AppBarButton`、`AppBarToggleButton` 和 `AppBarSeparator`。

## CommandBar 属性

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `PrimaryCommands` | `IList<ICommandBarElement>?` | `null` | 直接显示在工具栏中的命令。 |
| `SecondaryCommands` | `IList<ICommandBarElement>?` | `null` | 显示在溢出弹出菜单中的命令。 |
| `Content` | `object?` | `null` | 显示在工具栏前端、命令按钮之前的自定义内容。 |
| `DefaultLabelPosition` | `CommandBarDefaultLabelPosition` | `Bottom` | 所有命令按钮标签的默认位置。见下方取值表。 |
| `IsDynamicOverflowEnabled` | `bool` | `false` | 为 `true` 时，无法适应可用宽度的主命令会自动移动到溢出弹出菜单中。 |
| `OverflowButtonVisibility` | `CommandBarOverflowButtonVisibility` | `Auto` | 控制溢出按钮何时显示。见下方取值表。 |
| `IsOpen` | `bool` | `false` | 溢出弹出菜单当前是否打开。可绑定。 |
| `IsSticky` | `bool` | `false` | 为 `true` 时，命令执行后溢出弹出菜单仍保持打开。 |
| `HasSecondaryCommands` | `bool` | computed | 只读。当溢出弹出菜单中至少包含一个命令时为 `true`。 |
| `IsOverflowButtonVisible` | `bool` | computed | 只读。当前是否渲染了溢出按钮。 |
| `VisiblePrimaryCommands` | `ReadOnlyObservableCollection<ICommandBarElement>` | computed | 只读。当前仍显示在工具栏中的主命令（未被移入溢出区）。 |
| `OverflowItems` | `ReadOnlyObservableCollection<ICommandBarElement>` | computed | 只读。当前显示在溢出弹出菜单中的项目（包括次级命令和任何动态溢出项）。 |

## CommandBar 事件

| 事件 | 说明 |
| ----- | ----------- |
| `Opening` | 在溢出弹出菜单即将打开时触发。 |
| `Opened` | 在溢出弹出菜单已经打开后触发。 |
| `Closing` | 在溢出弹出菜单即将关闭时触发。 |
| `Closed` | 在溢出弹出菜单已经关闭后触发。 |

## DefaultLabelPosition 取值

| 值 | 说明 |
| ----- | ----------- |
| `Bottom` | 默认值。标签显示在图标下方。 |
| `Right` | 标签显示在图标右侧。适用于横向空间较充足的工具栏。 |
| `Collapsed` | 不显示标签，仅显示图标。为了无障碍，请添加 `ToolTip.Tip`。 |

## OverflowButtonVisibility 取值

| 值 | 说明 |
| ----- | ----------- |
| `Auto` | 默认值。只有在 `SecondaryCommands` 非空或启用了动态溢出时才显示溢出按钮。 |
| `Visible` | 始终显示溢出按钮，即使没有次级命令。 |
| `Collapsed` | 永不显示溢出按钮。 |

## CommandBarButton 属性

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Label` | `string?` | `null` | 显示在图标下方或旁边的文本标签，具体位置取决于 `DefaultLabelPosition`。 |
| `Icon` | `object?` | `null` | 图标内容，通常是 `PathIcon`。 |
| `IsCompact` | `bool` | `false` | 为 `true` 时，标签会被隐藏，按钮占用空间更小。通常由 `CommandBar` 自动设置。 |
| `LabelPosition` | `CommandBarDefaultLabelPosition` | `Bottom` | 单个按钮的标签位置覆盖设置。通常由 `CommandBar` 从 `DefaultLabelPosition` 自动设置，但也可以手动设置以覆盖单个按钮。 |
| `DynamicOverflowOrder` | `int` | `0` | 动态溢出的优先级。值越高的项目越先移入溢出区；相同值的项目会一起溢出。 |
| `IsInOverflow` | `bool` | `false` | 只读。当按钮当前显示在溢出弹出菜单中时为 `true`。 |

`AppBarButton` 也支持 `Command`、`CommandParameter` 以及从 `Button` 继承而来的标准 `Click` 事件。

## CommandBarToggleButton 属性

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Label` | `string?` | `null` | 文本标签。 |
| `Icon` | `object?` | `null` | 图标内容，通常是 `PathIcon`。 |
| `IsChecked` | `bool?` | `false` | 按钮是否处于选中状态。可空，以支持三态。 |
| `IsCompact` | `bool` | `false` | 为 `true` 时，标签会被隐藏。 |
| `LabelPosition` | `CommandBarDefaultLabelPosition` | `Bottom` | 单个按钮的标签位置覆盖设置。通常由 `CommandBar` 从 `DefaultLabelPosition` 自动设置，但也可以手动设置以覆盖单个按钮。 |
| `DynamicOverflowOrder` | `int` | `0` | 动态溢出的优先级。 |
| `IsInOverflow` | `bool` | `false` | 只读。当显示在溢出区时为 `true`。 |

`AppBarToggleButton` 在状态变化时会触发 `IsCheckedChanged`。

## CommandBarSeparator

`CommandBarSeparator` 会在命令分组之间绘制一条可视分隔线。它没有标签也没有图标。它实现了 `ICommandBarElement`，因此可以放在 `PrimaryCommands` 或 `SecondaryCommands` 中。

## 示例

### XAML 中的基础 CommandBar

```xml
<CommandBar>
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Add" Icon="{StaticResource AddIcon}" />
        <CommandBarButton Label="Edit" Icon="{StaticResource EditIcon}" />
        <CommandBarButton Label="Delete" Icon="{StaticResource DeleteIcon}" />
    </CommandBar.PrimaryCommands>
    <CommandBar.SecondaryCommands>
        <AppBarButton Label="Export" Click="OnExportClick" />
        <AppBarButton Label="Print"  Click="OnPrintClick" />
    </CommandBar.SecondaryCommands>
</CommandBar>
```

### 带分隔符的主命令与次级命令

使用 `CommandBarSeparator` 对命令进行分组，并将不太常用的操作放入 `SecondaryCommands`：

```xml
<CommandBar>
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Cut" Icon="{StaticResource CutIcon}" />
        <CommandBarButton Label="Copy" Icon="{StaticResource CopyIcon}" />
        <CommandBarButton Label="Paste" Icon="{StaticResource PasteIcon}" />
        <CommandBarSeparator />
        <CommandBarButton Label="Undo" Icon="{StaticResource UndoIcon}" />
        <CommandBarButton Label="Redo" Icon="{StaticResource RedoIcon}" />
    </CommandBar.PrimaryCommands>
    <CommandBar.SecondaryCommands>
        <CommandBarButton Label="Select All" />
        <CommandBarButton Label="Find and Replace" />
        <CommandBarSeparator />
        <CommandBarButton Label="Settings" />
    </CommandBar.SecondaryCommands>
</CommandBar>
```

<Image light={CommandBarSecondaryCommandsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### Custom Content Area

The `Content` property places custom content at the leading edge of the bar, before the commands:

```xml
<CommandBar>
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Back" Icon="{StaticResource BackIcon}" />
        <CommandBarButton Label="Forward" Icon="{StaticResource ForwardIcon}" />
    </CommandBar.PrimaryCommands>
    <CommandBar.Content>
        <TextBlock Text="Document Editor"
                   VerticalAlignment="Center"
                   Margin="8,0" />
    </CommandBar.Content>
    <CommandBar.PrimaryCommands>
        <AppBarButton Label="Save"  Click="OnSaveClick" />
        <AppBarButton Label="Undo"  Click="OnUndoClick" />
    </CommandBar.PrimaryCommands>
</CommandBar>
```

<Image light={CommandBarContentScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### Label Positions

```xml
<!-- Default: label below icon -->
<CommandBar DefaultLabelPosition="Bottom">
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Add" Icon="{StaticResource AddIcon}" />
        <CommandBarButton Label="Edit" Icon="{StaticResource EditIcon}" />
    </CommandBar.PrimaryCommands>
</CommandBar>

<!-- Label to the right of the icon -->
<CommandBar DefaultLabelPosition="Right">
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Add" Icon="{StaticResource AddIcon}" />
        <CommandBarButton Label="Edit" Icon="{StaticResource EditIcon}" />
    </CommandBar.PrimaryCommands>
</CommandBar>

<!-- Icon only, no label -->
<CommandBar DefaultLabelPosition="Collapsed" OverflowButtonVisibility="Collapsed">
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Add" Icon="{StaticResource AddIcon}" />
        <CommandBarButton Label="Edit" Icon="{StaticResource EditIcon}" />
    </CommandBar.PrimaryCommands>
</CommandBar>
```

When using `Collapsed`, always add `ToolTip.Tip` to preserve accessibility.

<Image light={CommandBarLabelBottomScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

<Image light={CommandBarLabelRightScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

```xml
<CommandBar IsDynamicOverflowEnabled="True">
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Important" Icon="{StaticResource StarIcon}" DynamicOverflowOrder="2" />
        <CommandBarButton Label="Less Important" Icon="{StaticResource InfoIcon}" DynamicOverflowOrder="0" />
        <CommandBarButton Label="Moderate" Icon="{StaticResource EditIcon}" DynamicOverflowOrder="1" />
    </CommandBar.PrimaryCommands>
</CommandBar>
```

In this example, "Less Important" overflows first (order 0), then "Moderate" (order 1), and "Important" overflows last (order 2).

### Dynamic Overflow

Enable `IsDynamicOverflowEnabled` so commands automatically move to overflow when the bar is too narrow. Use `DynamicOverflowOrder` to control which commands move first. Higher values overflow first:

```xml
<!-- Always show the overflow button -->
<CommandBar OverflowButtonVisibility="Visible">
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Save" Icon="{StaticResource SaveIcon}" />
    </CommandBar.PrimaryCommands>
</CommandBar>

<!-- Never show the overflow button -->
<CommandBar OverflowButtonVisibility="Collapsed">
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Save" Icon="{StaticResource SaveIcon}" />
    </CommandBar.PrimaryCommands>
</CommandBar>
```

### Overflow Button Visibility

```xml
<CommandBar IsSticky="True">
    <CommandBar.SecondaryCommands>
        <CommandBarToggleButton Label="Bold" Icon="{StaticResource BoldIcon}" />
        <CommandBarToggleButton Label="Italic" Icon="{StaticResource ItalicIcon}" />
        <CommandBarToggleButton Label="Underline" Icon="{StaticResource UnderlineIcon}" />
    </CommandBar.SecondaryCommands>
</CommandBar>
```

### Controlling overflow programmatically

```xml
<CommandBar IsOpen="{Binding IsOverflowOpen}">
    <CommandBar.SecondaryCommands>
        <CommandBarButton Label="Option A" />
        <CommandBarButton Label="Option B" />
    </CommandBar.SecondaryCommands>
</CommandBar>
```

### Sticky Overflow Menu

Keep the overflow popup open after the user invokes a command:

```csharp
commandBar.IsSticky = true;
```

### Controlling Overflow Programmatically

```csharp
// Open the overflow popup
commandBar.IsOpen = true;

// Close it
commandBar.IsOpen = false;
```

```xml
<CommandBar Opened="OnCommandBarOpened" Closed="OnCommandBarClosed">
    <CommandBar.SecondaryCommands>
        <CommandBarButton Label="Help" />
    </CommandBar.SecondaryCommands>
</CommandBar>
```

### CommandBar in a ContentPage

Place the bar in `ContentPage.TopCommandBar` so it sits above the page content:

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.EditorPage"
             Header="Editor">

    <ContentPage.TopCommandBar>
        <CommandBar>
            <CommandBar.PrimaryCommands>
                <CommandBarButton Label="Refresh" Icon="{StaticResource RefreshIcon}" />
                <CommandBarButton Label="Filter" Icon="{StaticResource FilterIcon}" />
            </CommandBar.PrimaryCommands>
            <CommandBar.SecondaryCommands>
                <AppBarButton Label="Find and Replace" Click="OnFindClick" />
                <AppBarButton Label="Export"           Click="OnExportClick" />
            </CommandBar.SecondaryCommands>
        </CommandBar>
    </ContentPage.TopCommandBar>

    <TextBox AcceptsReturn="True" Text="{Binding Content}" />

</ContentPage>
```

### CommandBar via NavigationPage Attached Property

When the page is inside a `NavigationPage`, use `NavigationPage.TopCommandBar` to place the bar inside the navigation bar area:

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             Header="Details">
    <NavigationPage.CommandBar>
        <CommandBar>
            <CommandBar.PrimaryCommands>
                <CommandBarButton Label="Share" Icon="{StaticResource ShareIcon}" />
                <CommandBarButton Label="Favorite" Icon="{StaticResource HeartIcon}" />
            </CommandBar.PrimaryCommands>
        </CommandBar>
    </NavigationPage.CommandBar>

</ContentPage>
```

### MVVM command binding

Bind `CommandBarButton` commands to your view model:

```xml
<CommandBar>
    <CommandBar.PrimaryCommands>
        <CommandBarButton Label="Save"
                      Icon="{StaticResource SaveIcon}"
                      Command="{Binding SaveCommand}" />
        <CommandBarButton Label="Delete"
                      Icon="{StaticResource DeleteIcon}"
                      Command="{Binding DeleteCommand}"
                      IsEnabled="{Binding HasSelection}">
            <AppBarButton.Icon>
                <PathIcon Data="M19,4H15.5L14.5..." />
            </AppBarButton.Icon>
        </AppBarButton>
    </CommandBar.PrimaryCommands>
</CommandBar>
```

### Toggle Button State Handling

```csharp
private void OnBoldChanged(object? sender, RoutedEventArgs e)
{
    if (sender is AppBarToggleButton btn)
    {
        bool isBold = btn.IsChecked == true;
        ApplyBold(isBold);
    }
}
```

### Toggle button state handling

Use `CommandBarToggleButton` to create togglable options. Bind the `IsChecked` property to track state:

```xml
<CommandBar>
    <CommandBar.PrimaryCommands>
        <CommandBarToggleButton Label="Bold"
                            Icon="{StaticResource BoldIcon}"
                            IsChecked="{Binding IsBold}" />
        <CommandBarToggleButton Label="Italic"
                            Icon="{StaticResource ItalicIcon}"
                            IsChecked="{Binding IsItalic}" />
        <CommandBarToggleButton Label="Underline"
                            Icon="{StaticResource UnderlineIcon}"
                            IsChecked="{Binding IsUnderline}" />
    </CommandBar.PrimaryCommands>
</CommandBar>
```

<Image light={CommandBarToggleButtonScreenshot} alt="CommandBar with toggle buttons" position="center" maxWidth={400} cornerRadius="true"/>

```csharp
[ObservableProperty]
private bool _isBold;

[ObservableProperty]
private bool _isItalic;

[ObservableProperty]
private bool _isUnderline;
```

## See also

- [API reference](/api/avalonia/controls/commandbar)
- [Source code](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/CommandBar/CommandBar.cs)
