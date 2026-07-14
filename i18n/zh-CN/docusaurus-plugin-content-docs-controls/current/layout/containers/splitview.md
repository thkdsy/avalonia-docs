---
id: splitview
title: SplitView
description: 了解如何使用 SplitView 控件在 Avalonia UI 中创建可折叠的侧边面板和导航侧栏。
doc-type: reference
---

import SplitViewCompactScreenshot from '/img/controls/splitview/splitview-expander.gif';

`SplitView` 表示一个由两部分组成的容器：主内容区域和侧边面板。主内容区域始终可见。侧边面板可以展开或折叠。折叠后的面板可以完全隐藏，也可以保留一小部分可见区域——例如，足以容纳几个图标按钮。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
| ------------------- | -------------------------------------------------------------------------------- |
| `PanePlacement` | 设置面板的位置：`Left`、`Right`、`Top` 或 `Bottom`。 |
| `IsPaneOpen` | 布尔值，默认是 true。面板当前是否处于打开状态？ |
| `DisplayMode` | 控制面板在打开和关闭状态下的绘制方式。见下文。 |
| `OpenPaneLength` | 定义面板在打开时的宽度（若为上/下放置则表示高度）。 |
| `CompactPaneLength` | 定义面板在关闭且显示模式为紧凑模式时的宽度（若为上/下放置则表示高度）。 |

显示模式属性控制面板在打开和关闭状态下的绘制方式。共有四种选项：

*   **Overlay**

    面板在打开前会被完全隐藏。打开后，面板会覆盖在内容区域之上。
*   **Inline**

    面板始终可见，宽度固定，并且不会覆盖内容区域。面板和内容区域会共同分配可用空间；如果容器宽度发生变化，变化的是内容区域的尺寸。
*   **Compact Overlay**

    在此模式下，面板始终会保留一小条可见区域，宽度刚好足以显示图标。默认的关闭状态面板宽度为 48px，可通过 `CompactPaneLength` 属性修改。面板打开后会覆盖内容区域。
*   **Compact Inline**

    在此模式下，面板始终会保留一小条可见区域，宽度刚好足以显示图标。默认的关闭状态面板宽度为 48px，可通过 `CompactPaneLength` 属性修改。面板打开后会压缩内容区域的大小。

## 示例

<XamlPreview>

```xml
<SplitView xmlns="https://github.com/avaloniaui"
           IsPaneOpen="True"
           DisplayMode="Inline"
           OpenPaneLength="100">
    <SplitView.Pane>
        <TextBlock Text="Pane"
                   FontSize="24"
                   VerticalAlignment="Center"
                   HorizontalAlignment="Center"/>
    </SplitView.Pane>

    <Grid>
        <TextBlock Text="Content"
                   FontSize="24"
                   VerticalAlignment="Center"
                   HorizontalAlignment="Center"/>
    </Grid>
</SplitView>
```

</XamlPreview>

## 紧凑显示模式

你可以将 MVVM 模式与 SplitView 控件及其某种紧凑显示模式结合使用，从而实现一种“工具面板”风格的 UI。面板关闭时仍会保留足够空间，用于显示一个可以重新打开面板的图标按钮。

<Image light={SplitViewCompactScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 导航侧栏模式

`SplitView` 的一个常见用途是创建带有图标按钮的可折叠导航侧栏：

```xml
<SplitView IsPaneOpen="{Binding IsPaneOpen}"
           DisplayMode="CompactInline"
           CompactPaneLength="48"
           OpenPaneLength="200">
    <SplitView.Pane>
        <StackPanel>
            <Button Content="☰" Command="{Binding TogglePaneCommand}"
                    Width="48" HorizontalAlignment="Left" />
            <ListBox ItemsSource="{Binding NavItems}"
                     SelectedItem="{Binding SelectedNavItem}"
                     Background="Transparent">
                <ListBox.ItemTemplate>
                    <DataTemplate>
                        <StackPanel Orientation="Horizontal" Spacing="12" Height="40">
                            <PathIcon Data="{Binding Icon}" Width="16" />
                            <TextBlock Text="{Binding Title}" VerticalAlignment="Center" />
                        </StackPanel>
                    </DataTemplate>
                </ListBox.ItemTemplate>
            </ListBox>
        </StackPanel>
    </SplitView.Pane>
    <SplitView.Content>
        <ContentControl Content="{Binding CurrentPage}" />
    </SplitView.Content>
</SplitView>
```

```csharp
[ObservableProperty]
private bool _isPaneOpen = true;

[RelayCommand]
private void TogglePane() => IsPaneOpen = !IsPaneOpen;
```

## 面板位置

你可以将面板放置在内容区域的任意一侧：

```xml
<!-- 面板放在右侧 -->
<SplitView PanePlacement="Right"
           IsPaneOpen="True"
           DisplayMode="Inline"
           OpenPaneLength="250">
```

`Top` 和 `Bottom` 放置方式会创建垂直分割布局，此时面板显示在内容区域的上方或下方。`OpenPaneLength` 和 `CompactPaneLength` 在这些方向下控制的是面板高度：

```xml
<!-- 面板放在顶部 -->
<SplitView PanePlacement="Top"
           IsPaneOpen="True"
           DisplayMode="Inline"
           OpenPaneLength="150">
    <SplitView.Pane>
        <TextBlock Text="Top pane" Margin="8" />
    </SplitView.Pane>
    <TextBlock Text="Main content" Margin="8" />
</SplitView>
```

## 另请参阅

- [SplitView API reference](/api/avalonia/controls/splitview)
- [`SplitView.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/SplitView/SplitView.cs)
