---
id: expander-how-to
title: "如何：使用 Expander"
description: 学习如何在 Avalonia 中使用 Expander 控件实现可折叠内容区、手风琴模式、动画过渡、状态绑定和自定义标题。
doc-type: how-to
---

本指南介绍常见的 [`Expander`](/api/avalonia/controls/expander) 使用场景，包括基础用法、手风琴模式、展开动画、状态绑定以及自定义标题。

## 基础 Expander

最简单的 `Expander` 用法，就是把你想显示或隐藏的内容放在一个可点击标题之后：

```xml
<Expander Header="Advanced Options">
    <StackPanel Spacing="8" Margin="8">
        <CheckBox Content="Enable logging" />
        <CheckBox Content="Verbose output" />
    </StackPanel>
</Expander>
```

`Expander` 只能直接承载一个子元素。如果你需要放多个控件，请先用 `StackPanel` 或 `Grid` 之类的布局面板把它们包起来。

## 初始展开

将 `IsExpanded` 设置为 `True`，让内容在控件首次加载时就显示出来：

```xml
<Expander Header="Details" IsExpanded="True">
    <TextBlock Text="This content is visible by default." TextWrapping="Wrap" />
</Expander>
```

:::tip
如果某部分内容大多数用户一开始就需要看到，但你又希望他们之后可以手动折叠以节省空间，那么“默认展开”的 `Expander` 就很适合这种场景。
:::

## 绑定 `IsExpanded`

你可以在视图模型中跟踪展开状态，这样 UI 的其他部分也能对其作出响应：

```csharp
[ObservableProperty]
private bool _showAdvanced;
```

```xml
<Expander Header="Advanced" IsExpanded="{Binding ShowAdvanced}">
    <StackPanel Spacing="8">
        <TextBox PlaceholderText="Custom path" />
    </StackPanel>
</Expander>
```

这个双向绑定会在用户打开或关闭 `Expander` 时，始终让视图模型属性保持同步。

## 展开方向

[`ExpandDirection`](/api/avalonia/controls/expanddirection) 属性用于控制内容相对于标题的展开方向：

```xml
<!-- 向上展开 -->
<Expander ExpandDirection="Up" Header="Details" VerticalAlignment="Bottom">
    <TextBlock Text="Content above the header" />
</Expander>

<!-- 向右展开 -->
<Expander ExpandDirection="Right" Header="More">
    <TextBlock Text="Content beside the header" />
</Expander>
```

| 值 | 说明 |
|---|---|
| `Down` | 内容显示在标题下方（默认）。 |
| `Up` | 内容显示在标题上方。 |
| `Left` | 内容显示在标题左侧。 |
| `Right` | 内容显示在标题右侧。 |

:::note
如果你使用 `Up`，请把 `Expander` 放在父容器底部（例如设置 `VerticalAlignment="Bottom"`），这样展开内容才有足够空间向上生长。`Left` 和 `Right` 的使用也同理，需要结合水平对齐一起考虑。
:::

## 带图标的自定义标题

通过 `Expander.Header`，你可以在标题区域中放置更丰富的内容：

```xml
<Expander>
    <Expander.Header>
        <StackPanel Orientation="Horizontal" Spacing="8">
            <PathIcon Data="{StaticResource settings_regular}" Width="16" />
            <TextBlock Text="Settings" VerticalAlignment="Center" />
        </StackPanel>
    </Expander.Header>
    <StackPanel Spacing="8" Margin="8">
        <TextBlock Text="Configuration options here" />
    </StackPanel>
</Expander>
```

由于 `Header` 的类型是 `object`，因此你可以为它赋予任意控件树。常见形式包括“图标 + 文本”、徽章、状态指示器等。

## 手风琴模式（始终只展开一个）

如果你希望任意时刻只有一个 `Expander` 保持展开，可以把每个 `IsExpanded` 绑定到视图模型中的同一个共享状态字段：

```csharp
public partial class AccordionViewModel : ObservableObject
{
    [ObservableProperty]
    private int _openSection = -1;

    public bool IsSection0Open
    {
        get => OpenSection == 0;
        set { if (value) OpenSection = 0; else if (OpenSection == 0) OpenSection = -1; }
    }

    public bool IsSection1Open
    {
        get => OpenSection == 1;
        set { if (value) OpenSection = 1; else if (OpenSection == 1) OpenSection = -1; }
    }

    public bool IsSection2Open
    {
        get => OpenSection == 2;
        set { if (value) OpenSection = 2; else if (OpenSection == 2) OpenSection = -1; }
    }

    partial void OnOpenSectionChanged(int value)
    {
        OnPropertyChanged(nameof(IsSection0Open));
        OnPropertyChanged(nameof(IsSection1Open));
        OnPropertyChanged(nameof(IsSection2Open));
    }
}
```

```xml
<StackPanel Spacing="4">
    <Expander Header="General" IsExpanded="{Binding IsSection0Open}">
        <TextBlock Text="General content" Margin="8" />
    </Expander>
    <Expander Header="Appearance" IsExpanded="{Binding IsSection1Open}">
        <TextBlock Text="Appearance content" Margin="8" />
    </Expander>
    <Expander Header="Advanced" IsExpanded="{Binding IsSection2Open}">
        <TextBlock Text="Advanced content" Margin="8" />
    </Expander>
</StackPanel>
```

当 `OpenSection` 为 `-1` 时，表示所有区域都处于折叠状态。用户一旦展开某个区域，之前已经展开的区域就会自动关闭。

## 内容动画过渡

你可以通过设置 `ContentTransition`，为展开和折叠添加更平滑的动画：

```xml
<Expander Header="Animated Section">
    <Expander.ContentTransition>
        <CrossFade Duration="0:0:0.2" />
    </Expander.ContentTransition>
    <StackPanel Spacing="8" Margin="8">
        <TextBlock Text="This content fades in and out." />
    </StackPanel>
</Expander>
```

除了 `CrossFade`，你也可以改用 `PageSlide`、`CompositePageTransition` 等其他过渡效果。完整内置选项请参阅 [Page transitions](/docs/graphics-animation/page-transitions)。

## 响应展开和折叠事件

你可以在 code-behind 中订阅 `IsExpandedChanged` 事件，以响应展开状态变化：

```csharp
private void Expander_IsExpandedChanged(object sender, RoutedEventArgs e)
{
    if (sender is Expander expander && expander.IsExpanded)
    {
        // 第一次展开时加载数据
    }
}
```

```xml
<Expander Header="Lazy Content"
          PropertyChanged="Expander_IsExpandedChanged" />
```

另外，你也可以在视图模型中使用属性变更回调来实现同样效果，而无需依赖 code-behind：

```csharp
[ObservableProperty]
private bool _isDetailsOpen;

partial void OnIsDetailsOpenChanged(bool value)
{
    if (value)
        LoadDetails();
}
```

:::tip
对于那些包含高成本渲染内容的 `Expander`，延迟加载是一种非常实用的模式。把真正的工作延后到用户实际展开该区域时再执行。
:::

## 设置 Expander 样式

### 移除边框

你可以移除默认边框和背景，从而创建更简洁的外观：

```xml
<Expander.Styles>
    <Style Selector="Expander">
        <Setter Property="BorderThickness" Value="0" />
        <Setter Property="Background" Value="Transparent" />
    </Style>
</Expander.Styles>
```

### 自定义展开图标

通过重写模板中的切换按钮，可以修改展开/折叠指示图标：

```xml
<Expander.Styles>
    <Style Selector="Expander /template/ ToggleButton#PART_toggle">
        <!-- 覆盖切换按钮的外观 -->
    </Style>
</Expander.Styles>
```

### 禁用状态

当你把 `IsEnabled="False"` 设置到某个 `Expander` 上时，标题就不再可交互，用户也无法再切换内容的展开状态。不过，在被禁用的那一刻，它当前的展开/折叠状态会被保留下来。

```xml
<Expander Header="Read-only section" IsEnabled="False" IsExpanded="True">
    <TextBlock Text="This section cannot be collapsed." Margin="8" />
</Expander>
```

## 关键属性参考

| 属性 | 类型 | 说明 |
|---|---|---|
| `Header` | `object` | 始终可见的标题区域内容。 |
| `IsExpanded` | `bool` | 内容区域是否可见，默认值为 `False`。 |
| `ExpandDirection` | `ExpandDirection` | 内容展开方向：`Down`、`Up`、`Left`、`Right`。默认值为 `Down`。 |
| `ContentTransition` | `IPageTransition` | 展开和折叠时使用的动画。 |
| `IsEnabled` | `bool` | 用户是否可以通过标题区域切换展开状态。 |

## 另请参阅

- [Expander control reference](/controls/layout/containers/expander)：完整属性与事件说明。
- [Page transitions](/docs/graphics-animation/page-transitions)：可用于内容动画的过渡类型。
- [Introduction to data binding](/docs/data-binding/introduction-to-data-binding)：将 `IsExpanded` 等属性绑定到视图模型的基础知识。
