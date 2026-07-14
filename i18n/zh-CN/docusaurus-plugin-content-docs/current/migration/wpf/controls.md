---
id: controls
title: 控件
description: WPF 与 Avalonia 控件在层级、命名和行为上的差异。
doc-type: migration
---

import RenderTransformOriginWpfScreenshot from '/img/guides/migration/wpf/rendertransformorigin-wpf.png';
import RenderTransformOriginAvaloniaScreenshot from '/img/guides/migration/wpf/rendertransformorigin-avalonia.png';

本页介绍从 WPF 迁移到 Avalonia 时最常遇到的关键控件差异，包括基类变化、控件重命名以及行为差异。

## UIElement 与 FrameworkElement

WPF 中的 `UIElement` 和 `FrameworkElement` 都是非模板化控件基类，大致对应 Avalonia 中的 `Control` 类。而 WPF 的 `Control` 类本身则是模板化控件，对应到 Avalonia 中则是 `TemplatedControl`。

- 在 WPF/UWP 中，如果你要创建新的模板化控件，通常会继承 `Control`；而在 Avalonia 中，你应当继承 `TemplatedControl`。
- 在 WPF/UWP 中，如果你要创建新的自绘控件，通常会继承 `FrameworkElement`；而在 Avalonia 中，你应当继承 `Control`。

总结如下：

* `UIElement` 🠞 `Control`
* `FrameworkElement`🠞 `Control`
* `Control` 🠞 `TemplatedControl`

## RenderTransform 与 RenderTransformOrigin

WPF 与 Avalonia 的 `RenderTransformOrigin` 默认值不同：如果你使用了 `RenderTransform`，请注意 Avalonia 中 `RenderTransformOrigin` 的默认值是 `RelativePoint.Center`，而在 WPF 中默认值是 `RelativePoint.TopLeft` \(0, 0\)。因此在 `Viewbox` 这类控件中，相同代码可能会产生不同的渲染结果：

**在 WPF 中：**
<Image light={RenderTransformOriginWpfScreenshot} alt="WPF" position="center" maxWidth={400} cornerRadius="true"/>

**在 Avalonia 中：**
<Image light={RenderTransformOriginAvaloniaScreenshot} alt="Avalonia" position="center" maxWidth={400} cornerRadius="true"/>

在 AvaloniaUI 中，如果你想得到与 WPF 相同的缩放效果，就需要显式将 `RenderTransformOrigin` 设为 Visual 的 TopLeft。

## Grid

在 Avalonia 中，可以使用字符串直接指定列和行定义，从而避免 WPF 中较为繁琐的语法：

```xml
<Grid ColumnDefinitions="Auto,*,32" RowDefinitions="*,Auto">
```

在 WPF 中，`Grid` 的一个常见用途是将两个控件叠放在一起。对于这种场景，在 Avalonia 中可以使用比 `Grid` 更轻量的 `Panel`。

## ToolTip

WPF 将 `ToolTip` 作为属性或子元素使用；Avalonia 则使用 `ToolTip.Tip` 附加属性：

```xml title="WPF"
<Button ToolTip="Save the document" Content="Save" />
```

```xml title="Avalonia"
<Button ToolTip.Tip="Save the document" Content="Save" />
```

## ItemsControl 与 ItemsSource

WPF 中可以直接设置 `ItemsControl.Items`。而在 Avalonia 中，数据绑定应使用 `ItemsSource`，或者直接在 XAML 中添加子元素：

```xml title="Avalonia"
<ListBox ItemsSource="{Binding MyItems}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

注意：在 Avalonia 中，虽然仍然使用同名的 `ItemsSource`，但 `Items` 本身是只读的，不能为它直接赋予一个新的集合。

## DataGrid

在 Avalonia 中，DataGrid 是一个单独的 NuGet 包：

```xml
<PackageReference Include="Avalonia.Controls.DataGrid" Version="$(AvaloniaVersion)" />
```

你还需要在 `App.axaml` 中引入 DataGrid 的主题：

```xml
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://Avalonia.Controls.DataGrid/Themes/Fluent.axaml" />
</Application.Styles>
```

## StatusBar

Avalonia 没有内置 `StatusBar` 控件。通常可以在窗口底部使用一个带样式的 `DockPanel` 或 `StackPanel` 来替代：

```xml
<DockPanel>
    <Border DockPanel.Dock="Bottom" Background="{DynamicResource SystemChromeLowColor}" Padding="8,4">
        <TextBlock Text="Ready" />
    </Border>
    <!-- Main content -->
</DockPanel>
```

## RichTextBox

Avalonia 没有内置 `RichTextBox`。如果需要富文本编辑，请使用诸如 AvalonEdit 之类的第三方控件。

## 另请参阅

- [WPF 到 Avalonia 速查表](/docs/migration/wpf/cheat-sheet)：所有控件映射的快速参考。
- [控件参考](/controls)：完整的 Avalonia 控件文档。
