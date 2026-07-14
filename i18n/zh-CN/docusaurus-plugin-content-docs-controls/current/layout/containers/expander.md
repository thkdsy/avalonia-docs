---
id: expander
title: Expander
description: 一个带有始终可见标题和可折叠内容区域的容器控件，可在打开和关闭之间切换。
doc-type: reference
---

[`Expander`](/api/avalonia/controls/expander) 控件具有一个始终可见的标题区域，以及一个可折叠的内容区域，该区域可包含单个子控件。用户点击标题即可切换内容的可见性。当你希望在不离开当前视图的情况下，让用户展开或隐藏补充信息（例如搜索筛选、高级设置或可选表单字段）时，这会非常有用。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
|---|---|---|
| `Header` | `object` | 显示在始终可见标题区域中的内容。可接受字符串、控件或数据模板。 |
| `IsExpanded` | `bool` | 内容区域当前是否可见。默认值为 `false`。 |
| [`ExpandDirection`](/api/avalonia/controls/expanddirection) | `ExpandDirection` | 内容展开的方向：`Down`（默认）、`Up`、`Left` 或 `Right`。 |
| `ContentTransition` | `IPageTransition` | 内容展开或折叠时播放的过渡动画。 |

## 事件

| 事件 | 说明 |
|---|---|
| `Expanding` | 当内容区域开始展开时触发。 |
| `Collapsed` | 当内容区域完成折叠时触发。 |

## 基本示例

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="5">
  <Expander VerticalAlignment="Top">
      <Expander.Header>
          Hidden Search
      </Expander.Header>
      <Grid RowDefinitions="*,*" ColumnDefinitions="150,*">
        <TextBlock Grid.Row="0" Grid.Column="0"
                   VerticalAlignment="Center">Search</TextBlock>
        <TextBox Grid.Row="0" Grid.Column="1"
                 PlaceholderText="Search text" Width="200" />
        <TextBlock Grid.Row="1" Grid.Column="0"
                   VerticalAlignment="Center">Case sensitive?</TextBlock>
        <CheckBox Grid.Row="1" Grid.Column="1" />
      </Grid>
  </Expander>
</UserControl>
```

</XamlPreview>

## 初始展开

将 `IsExpanded` 设置为 `True`，即可在控件首次加载时显示内容：

```xml
<Expander Header="Details" IsExpanded="True">
    <TextBlock Text="This content is visible by default." TextWrapping="Wrap" />
</Expander>
```

## 展开方向

你可以通过设置 `ExpandDirection` 来控制内容区域展开的方向。默认值为 `Down`，但你也可以使用 `Up`、`Left` 或 `Right`：

```xml
<Expander ExpandDirection="Up" Header="Expand Upward" VerticalAlignment="Bottom">
    <TextBlock Text="Content above the header" />
</Expander>
```

当你使用 `Left` 或 `Right` 时，标题会旋转，而内容会水平展开。这对于侧边面板或工具抽屉会很有用。

## 自定义标题内容

`Header` 属性接受任意内容，而不仅仅是字符串。你可以用它来创建带图标、徽章或其他控件的丰富标题：

```xml
<Expander>
    <Expander.Header>
        <StackPanel Orientation="Horizontal" Spacing="8">
            <PathIcon Data="{StaticResource settings_regular}" Width="16" />
            <TextBlock Text="Settings" VerticalAlignment="Center" />
        </StackPanel>
    </Expander.Header>
    <StackPanel Spacing="8" Margin="8">
        <CheckBox Content="Enable notifications" />
        <CheckBox Content="Auto-save" />
    </StackPanel>
</Expander>
```

## 绑定 `IsExpanded`

你可以将展开状态绑定到视图模型属性，以便通过程序来控制。这允许你在代码中展开或折叠该区域：

```xml
<Expander Header="Advanced" IsExpanded="{Binding ShowAdvanced}">
    <TextBox PlaceholderText="Custom path" />
</Expander>
```

```csharp
public class MyViewModel : ViewModelBase
{
    private bool _showAdvanced;

    public bool ShowAdvanced
    {
        get => _showAdvanced;
        set => this.RaiseAndSetIfChanged(ref _showAdvanced, value);
    }
}
```

## 内容过渡动画

你可以使用页面过渡动画来为展开和折叠动作添加动画效果。通过设置 `ContentTransition` 属性来控制该动画：

```xml
<Expander Header="Animated Section">
    <Expander.ContentTransition>
        <CrossFade Duration="0:0:0.2" />
    </Expander.ContentTransition>
    <TextBlock Text="Fades in and out" />
</Expander>
```

## 嵌套 Expander

你可以嵌套 `Expander` 控件，以创建可折叠的层级结构。只需将内层 `Expander` 放到外层 `Expander` 的内容中即可：

```xml
<Expander Header="General Settings" IsExpanded="True">
    <StackPanel Spacing="8" Margin="8">
        <CheckBox Content="Enable dark mode" />
        <Expander Header="Advanced">
            <StackPanel Spacing="8" Margin="8">
                <CheckBox Content="Hardware acceleration" />
                <CheckBox Content="Verbose logging" />
            </StackPanel>
        </Expander>
    </StackPanel>
</Expander>
```

:::tip
在嵌套 expander 时，请尽量保持层级较浅（最多两层），以免让用户感到界面过于复杂。
:::

## 无障碍支持

`Expander` 通过其 `ExpanderAutomationPeer` 提供内置的无障碍支持。屏幕阅读器会将该控件识别为可展开或可折叠，并朗读其当前状态。展开和折叠动作会通过 `IExpandCollapseProvider` 模式暴露给辅助技术，从而允许用户在不使用指针的情况下切换内容。

为了提升应用的无障碍性，请将 `Header` 设置为有意义的标签，以便屏幕阅读器能够描述每个可折叠区域的用途。

## 另请参阅

- [Expander API reference](/api/avalonia/controls/expander)
- [`Expander.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Expander.cs)
- [SplitView](/controls/layout/containers/splitview)
- [GroupBox](/controls/layout/containers/groupbox)
