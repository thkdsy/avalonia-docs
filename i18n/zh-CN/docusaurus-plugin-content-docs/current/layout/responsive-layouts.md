---
id: responsive-layouts
title: 响应式布局
description: 使用容器查询、设备形态扩展和可回流面板来创建可适应不同尺寸的布局。
doc-type: explanation
---

Avalonia 提供了多种技术，用于构建能够在可用空间变化时自适应的布局。你可以根据容器大小、设备类型，或可回流面板的尺寸变化来调整界面。本页将解释每种方案及其适用场景。

## 方案总览

| 技术 | 响应对象 | 生效时机 | 最适合的场景 |
|-----------|-------------|----------|----------|
| [容器查询](#container-queries) | 祖先控件的尺寸 | 实时，随控件缩放更新 | 出现在不同宽度面板中的可复用组件 |
| [`OnFormFactor`](#onformfactor) | 设备类型（桌面、移动端） | 启动时一次性解析 | 平台特定的布局差异 |
| [可回流面板](#reflowing-panels) | 可用宽度 | 实时，随面板缩放更新 | 卡片网格与流式内容 |
| [断点视图模型](#breakpoint-view-models) | 窗口宽度（或任意测量值） | 实时，通过属性变化 | 由代码驱动的复杂多属性过渡 |

## 容器查询

容器查询允许你在祖先控件达到特定尺寸时激活样式。由于查询针对的是控件而不是窗口本身，因此同一个组件无论出现在全宽页面、狭窄侧边栏还是对话框中，都能正确地自适应。

### 声明容器

通过设置 `Container.Name` 和 `Container.Sizing` 附加属性，可以将任意祖先元素标记为容器：

```xml
<Border Container.Name="main"
        Container.Sizing="Width">
    <!-- child content here -->
</Border>
```

`Container.Sizing` 用于决定需要跟踪哪些维度：

| 值 | 跟踪的维度 |
|-------|--------------------|
| `Normal` | 不跟踪（默认） |
| `Width` | 仅宽度 |
| `Height` | 仅高度 |
| `WidthAndHeight` | 同时跟踪宽度和高度 |

### 编写容器查询

`ContainerQuery` 元素需要放在某个容器祖先控件的 `Styles` 集合中。当查询条件满足时，它会激活其内部的子样式：

```xml
<Window>
    <Window.Styles>
        <ContainerQuery Name="main" Query="max-width:600">
            <Style Selector="StackPanel#sidebar">
                <Setter Property="IsVisible" Value="False" />
            </Style>
        </ContainerQuery>
    </Window.Styles>

    <Grid ColumnDefinitions="200,*">
        <StackPanel x:Name="sidebar" Grid.Column="0">
            <!-- sidebar content -->
        </StackPanel>
        <ContentControl Grid.Column="1"
                        Content="{Binding CurrentPage}" />
    </Grid>
</Window>
```

在这个示例中，当名为 `main` 的容器宽度小于或等于 600 像素时，侧边栏会被隐藏。

### 使用断点调整布局结构

你可以在同一个容器上使用多个容器查询来定义不同断点层级。下面的示例会在容器变宽时动态改变 `UniformGrid` 的列数：

```xml
<Panel Container.Name="content" Container.Sizing="Width">
    <Panel.Styles>
        <ContainerQuery Name="content" Query="max-width:400">
            <Style Selector="UniformGrid#cards">
                <Setter Property="Columns" Value="1" />
            </Style>
        </ContainerQuery>
        <ContainerQuery Name="content" Query="min-width:400">
            <Style Selector="UniformGrid#cards">
                <Setter Property="Columns" Value="2" />
            </Style>
        </ContainerQuery>
        <ContainerQuery Name="content" Query="min-width:800">
            <Style Selector="UniformGrid#cards">
                <Setter Property="Columns" Value="3" />
            </Style>
        </ContainerQuery>
    </Panel.Styles>

    <UniformGrid x:Name="cards">
        <!-- card items -->
    </UniformGrid>
</Panel>
```

### 自定义非布局属性

容器查询并不局限于布局属性。你可以调整任何 `Style` 可设置的属性，包括字体大小、间距、可见性和颜色：

```xml
<Panel Container.Name="content" Container.Sizing="Width">
    <Panel.Styles>
        <!-- 默认标题字号 -->
        <Style Selector="TextBlock.heading">
            <Setter Property="FontSize" Value="24" />
        </Style>

        <!-- 当容器较窄时使用更小的标题字号 -->
        <ContainerQuery Name="content" Query="max-width:500">
            <Style Selector="TextBlock.heading">
                <Setter Property="FontSize" Value="18" />
            </Style>
            <Style Selector="StackPanel.toolbar">
                <Setter Property="Orientation" Value="Vertical" />
            </Style>
        </ContainerQuery>
    </Panel.Styles>

    <StackPanel>
        <TextBlock Classes="heading" Text="Dashboard" />
        <StackPanel Classes="toolbar" Orientation="Horizontal" Spacing="8">
            <Button Content="New" />
            <Button Content="Refresh" />
        </StackPanel>
    </StackPanel>
</Panel>
```

### 组合查询

可以在单个查询中使用 `and`（必须全部满足）或 `,`（满足任意一个即可）来组合多个条件：

```xml
<!-- 两个条件都必须满足 -->
<ContainerQuery Name="main" Query="min-width:400 and max-width:800">
    <!-- 适用于中等宽度的样式 -->
</ContainerQuery>

<!-- 满足任意一个条件即可 -->
<ContainerQuery Name="main" Query="max-width:300,min-height:600">
    <!-- 适用于狭窄或较高容器的样式 -->
</ContainerQuery>
```

完整的查询语法、可用查询类型和限制条件，请参阅 [容器查询](/docs/styling/container-queries)。

:::tip
当 `TopLevel`（也就是你的窗口或主视图）被设置为容器时，容器查询的行为就类似于 CSS 媒体查询，会直接响应窗口本身的尺寸变化。
:::

## OnFormFactor

`OnFormFactor` 标记扩展会根据设备类型选择对应的值。它只会在启动时解析一次，因此不会在运行时对窗口大小变化作出响应。

### 设备形态取值

| 参数 | 匹配对象 | 常见平台 |
|-----------|---------|-------------------|
| `Desktop` | 桌面系统 | Windows、macOS、Linux |
| `Mobile` | 移动系统 | iOS、Android |
| `TV` | 电视系统 | tvOS、Android TV |
| `Default` | 当前设备形态未显式指定时的回退值 | 任意 |

如果当前设备形态不匹配任何已指定参数，则会使用 `Default` 值。如果没有设置 `Default`，该属性将得到其类型的默认值。

```xml
<Grid ColumnDefinitions="{OnFormFactor Desktop='250,*', Mobile='*'}">
    <Border Grid.Column="0"
            IsVisible="{OnFormFactor Desktop=True, Mobile=False}">
        <ListBox ItemsSource="{Binding MenuItems}" />
    </Border>
    <ContentControl Grid.Column="{OnFormFactor Desktop=1, Mobile=0}"
                    Content="{Binding CurrentPage}" />
</Grid>
```

当桌面端与移动端在布局结构上存在明显差异，且你不需要响应运行时窗口缩放时，适合使用 `OnFormFactor`。如果布局必须在用户拖动窗口尺寸时实时自适应，请改用容器查询。

### OnPlatform

相关的 `OnPlatform` 标记扩展是根据操作系统而不是设备类型来选择值的。它同样只会在启动时解析一次。

| 参数 | 匹配对象 |
|-----------|---------|
| `Windows` | Windows |
| `macOS` | macOS |
| `Linux` | Linux |
| `iOS` | iOS |
| `Android` | Android |
| `Browser` | 浏览器中的 WebAssembly（WASM） |
| `Default` | 当前平台未显式指定时的回退值 |

```xml
<TextBlock FontFamily="{OnPlatform macOS='San Francisco',
                                   Windows='Segoe UI',
                                   Default='Inter'}" />
```

`OnFormFactor` 和 `OnPlatform` 的用途不同。`OnFormFactor` 适用于不同设备类别（如桌面端与移动端）之间的结构性布局差异；`OnPlatform` 则适用于平台特定调整，例如原生字体族或操作系统专属样式。

## 可回流面板

某些面板会根据可用空间自动重新排布其子元素，而不需要编写查询或代码。

**`WrapPanel`** 会将子元素按行排列，并在到达面板边缘时自动换到下一行：

```xml
<WrapPanel Orientation="Horizontal">
    <Button Content="One" Margin="4" />
    <Button Content="Two" Margin="4" />
    <Button Content="Three" Margin="4" />
    <!-- 当面板过窄时自动换到下一行 -->
</WrapPanel>
```

**`UniformGridLayout`**（与 `ItemsRepeater` 配合使用）会根据可用宽度和最小项尺寸自动计算列数：

```xml
<ItemsRepeater ItemsSource="{Binding Cards}">
    <ItemsRepeater.Layout>
        <UniformGridLayout MinItemWidth="280"
                           MinItemHeight="200"
                           MinColumnSpacing="12"
                           MinRowSpacing="12" />
    </ItemsRepeater.Layout>
    <ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <Border Padding="16" CornerRadius="8"
                    Background="White"
                    BorderBrush="#E5E7EB" BorderThickness="1">
                <TextBlock Text="{Binding Title}" />
            </Border>
        </DataTemplate>
    </ItemsRepeater.ItemTemplate>
</ItemsRepeater>
```

当你希望内容自动流式排布，而不想显式定义断点时，这些面板是非常合适的选择。

## 断点视图模型

当你的响应式逻辑涉及多个需要联动的属性变化，或者条件不仅仅取决于尺寸（例如同时考虑方向和平台）时，可以在视图模型中监听窗口宽度，并为不同断点暴露布尔属性：

```csharp
public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private bool _isCompact;

    [ObservableProperty]
    private bool _isWide;

    public void UpdateLayout(double windowWidth)
    {
        IsCompact = windowWidth < 640;
        IsWide = windowWidth >= 1024;
    }
}
```

Call `UpdateLayout` from the window's size-changed handler:

```csharp
protected override void OnSizeChanged(SizeChangedEventArgs e)
{
    base.OnSizeChanged(e);
    if (DataContext is MainViewModel vm)
        vm.UpdateLayout(e.NewSize.Width);
}
```

Then bind layout properties to the breakpoint flags:

```xml
<StackPanel IsVisible="{Binding IsCompact}" Spacing="8">
    <views:SidebarView />
    <views:ContentView />
</StackPanel>

<Grid IsVisible="{Binding !IsCompact}" ColumnDefinitions="280,*">
    <views:SidebarView Grid.Column="0" />
    <views:ContentView Grid.Column="1" />
</Grid>
```

This approach provides full programmatic control but requires code-behind or view model wiring. Prefer container queries when your transitions are purely size-based and can be expressed in XAML.

## Choosing an approach

Use the following decision process to select the right technique:

1. **Does your component need to adapt based on its own size (not the window)?** Use container queries. This keeps the component self-contained and reusable.
2. **Are desktop and mobile layouts structurally different, with no need for live resizing?** Use `OnFormFactor`.
3. **Do you have a collection of items that should reflow into rows?** Use `WrapPanel` or `UniformGridLayout`.
4. **Does your transition logic involve multiple conditions, platform checks, or non-size triggers?** Use breakpoint view models.

You can combine these techniques. For example, use `OnFormFactor` for a top-level structural difference (sidebar vs. bottom tabs), then use container queries within individual panels so they adapt to their actual rendered size.

## See also

- [Container queries](/docs/styling/container-queries): Full query syntax, container sizing modes, and restrictions.
- [How to: Build responsive layouts](/docs/how-to/responsive-layout-how-to): Step-by-step recipes for common responsive patterns.
- [Layout](/docs/layout): How the Avalonia measure and arrange system works.
- [Choosing a layout panel](/docs/layout/choosing-a-layout-panel): Picking the right panel for your scenario.
- [`OnFormFactorExtension` API reference](/api/avalonia/markup/xaml/markupextensions/onformfactorextension)
- [`OnPlatformExtension` API reference](/api/avalonia/markup/xaml/markupextensions/onplatformextension)
