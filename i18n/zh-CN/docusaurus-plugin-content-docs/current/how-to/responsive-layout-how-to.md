---
id: responsive-layout-how-to
title: "如何：构建响应式布局"
description: 创建能够适应不同窗口尺寸和设备形态的 Avalonia 布局。
doc-type: how-to
---

本指南介绍如何创建能够适配不同窗口尺寸和设备形态的布局。你将学习如何使用设备形态标记扩展、容器查询、基于断点的视图模型，以及可重排的项目布局，从而构建既适用于桌面端又适用于移动端的 UI。

## 自适应 Grid 列布局

使用 `OnFormFactor` 标记扩展，可以根据设备类型切换布局结构。下面这个示例中，桌面端会显示一个带侧边栏的双列 Grid，而移动端则切换为单列布局：

```xml
<Grid ColumnDefinitions="{OnFormFactor Desktop='250,*', Mobile='*'}">
    <Border Grid.Column="0" Background="#F3F4F6"
            IsVisible="{OnFormFactor Desktop=True, Mobile=False}">
        <!-- 侧边栏：桌面端可见，移动端隐藏 -->
        <ListBox ItemsSource="{Binding MenuItems}" />
    </Border>

    <ContentControl Grid.Column="{OnFormFactor Desktop=1, Mobile=0}"
                    Content="{Binding CurrentPage}" />
</Grid>
```

`OnFormFactor` 会在启动时解析，因此当你在运行时调整窗口大小时，它的值不会自动变化。如果你希望布局能响应实时尺寸变化，请改用容器查询或基于断点的方案。

## 容器查询

容器查询是根据控件自身的实际渲染尺寸，而不是整个窗口尺寸，来适配布局的。这非常适合那些可能出现在不同宽度区域中的可复用组件。

下面的示例会根据父级 `Border` 的宽度，在垂直和水平 `StackPanel` 布局之间切换：

```xml
<Border>
    <Border.Styles>
        <!-- 容器较窄时使用垂直布局 -->
        <Style Selector="Border[Width<400] > StackPanel">
            <Setter Property="Orientation" Value="Vertical" />
        </Style>
        <!-- 容器较宽时使用水平布局 -->
        <Style Selector="Border[Width>=400] > StackPanel">
            <Setter Property="Orientation" Value="Horizontal" />
        </Style>
    </Border.Styles>

    <StackPanel Spacing="8">
        <TextBlock Text="Label" />
        <TextBox Text="{Binding Value}" />
    </StackPanel>
</Border>
```

See [Container queries](/docs/styling/container-queries) for the full syntax and named-container support.

## 基于断点的布局

当你需要对布局切换做更细粒度控制时，可以在视图模型中监听窗口宽度，并以此实现断点。常见做法是：为每个断点层级定义布尔属性，再让 XAML 绑定到这些属性：

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

你可以在窗口的 `OnSizeChanged` 重写方法中调用 `UpdateLayout`，从而在用户调整窗口大小时保持这些属性同步：

```csharp
// 在 MainWindow 的 code-behind 中
protected override void OnSizeChanged(SizeChangedEventArgs e)
{
    base.OnSizeChanged(e);
    if (DataContext is MainViewModel vm)
        vm.UpdateLayout(e.NewSize.Width);
}
```

在 AXAML 中，你可以通过把 `IsVisible` 绑定到这些断点属性，在紧凑布局和宽布局之间切换：

```xml
<Grid>
    <!-- 紧凑布局：单列 -->
    <StackPanel IsVisible="{Binding IsCompact}" Spacing="8">
        <views:SidebarView />
        <views:ContentView />
    </StackPanel>

    <!-- 宽布局：双列 -->
    <Grid IsVisible="{Binding !IsCompact}" ColumnDefinitions="280,*">
        <views:SidebarView Grid.Column="0" />
        <views:ContentView Grid.Column="1" />
    </Grid>
</Grid>
```

这种方式让你拥有完全的程序化控制能力，特别适合那些不仅仅依赖“宽度阈值”的布局逻辑，例如同时结合屏幕方向和平台判断的场景。

## 使用 `SplitView` 实现可折叠侧边栏

`SplitView` 控件内置了可折叠侧栏模式。你可以把 `DisplayMode` 设为 `CompactInline`，让侧栏在折叠时缩成一条只显示图标的窄条，而当 `IsPaneOpen` 切换为打开时，再展开显示文字标签：

```xml
<SplitView IsPaneOpen="{Binding IsSidebarOpen}"
           DisplayMode="CompactInline"
           CompactPaneLength="48"
           OpenPaneLength="250">
    <SplitView.Pane>
        <StackPanel>
            <Button Content="☰" Command="{Binding ToggleSidebarCommand}"
                    HorizontalAlignment="Left" Width="48" />
            <ListBox ItemsSource="{Binding MenuItems}"
                     SelectedItem="{Binding SelectedMenuItem}">
                <ListBox.ItemTemplate>
                    <DataTemplate>
                        <StackPanel Orientation="Horizontal" Spacing="12">
                            <PathIcon Data="{Binding Icon}" Width="16" />
                            <TextBlock Text="{Binding Title}" />
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

你还可以把 `IsPaneOpen` 绑定到断点属性上，让侧边栏在宽屏时自动展开，在窄屏时自动折叠。

## 响应式卡片网格

通过 `ItemsRepeater` 搭配 `UniformGridLayout`，可以构建一个会随着可用宽度变化而自动重排的卡片网格。你只需设置 `MinItemWidth` 和 `MinItemHeight` 来定义卡片的最小尺寸，`UniformGridLayout` 会自动计算列数：

```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{Binding Cards}">
        <ItemsRepeater.Layout>
            <UniformGridLayout MinItemWidth="280" MinItemHeight="200"
                               MinColumnSpacing="12" MinRowSpacing="12" />
        </ItemsRepeater.Layout>
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <Border Background="White" CornerRadius="8" Padding="16"
                        BorderBrush="#E5E7EB" BorderThickness="1">
                    <StackPanel Spacing="8">
                        <TextBlock Text="{Binding Title}" FontWeight="Bold" />
                        <TextBlock Text="{Binding Description}"
                                   TextWrapping="Wrap" Foreground="Gray" />
                    </StackPanel>
                </Border>
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

如果你需要的是一个更简单、且不要求虚拟化的自动换行容器，也可以改用 `WrapPanel`。

## 平台特定间距

使用 `OnFormFactor`，可以针对不同平台调整间距、边距和字体大小。通常来说，移动端界面会更适合更大的触控目标和略大一些的文字：

```xml
<StackPanel Spacing="{OnFormFactor Desktop=8, Mobile=12}"
            Margin="{OnFormFactor Desktop='16', Mobile='8'}">
    <TextBlock Text="Content" FontSize="{OnFormFactor Desktop=14, Mobile=16}" />
</StackPanel>
```

## 自适应字体大小

你可以使用 [container queries](/docs/styling/container-queries)，根据祖先容器的尺寸来缩放文本。先在父元素上通过 `Container.Name` 和 `Container.Sizing` 声明一个容器，再通过 `ContainerQuery` 为不同宽度设置不同的字体大小：

```xml
<Panel Container.Name="content" Container.Sizing="Width">
    <Panel.Styles>
        <Style Selector="TextBlock.title">
            <Setter Property="FontSize" Value="24" />
        </Style>

        <!-- 容器较窄时使用更小的标题字号 -->
        <ContainerQuery Name="content" Query="max-width:500">
            <Style Selector="TextBlock.title">
                <Setter Property="FontSize" Value="18" />
            </Style>
        </ContainerQuery>
    </Panel.Styles>

    <TextBlock Classes="title" Text="Responsive heading" />
</Panel>
```

这种技术可以让你的排版具备响应能力，而无需依赖窗口级断点。即使控件位于分栏面板或对话框中，文本也能正确适配。

## 另请参阅

- [Container queries](/docs/styling/container-queries)：基于容器大小的响应式样式能力。
- [Layout](/docs/layout)：Avalonia 布局系统概览。
- [Grid how-to](/docs/how-to/grid-how-to)：Grid 布局模式。
- [Cross-platform architecture](/docs/fundamentals/cross-platform-architecture)：平台检测与分支处理。
