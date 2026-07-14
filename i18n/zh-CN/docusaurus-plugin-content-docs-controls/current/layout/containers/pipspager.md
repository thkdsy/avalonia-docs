---
id: pipspager
title: PipsPager
description: Avalonia 中 PipsPager 控件的参考文档，用于显示支持分页导航的交互式点状指示器，并可选择显示上一页/下一页按钮。
doc-type: reference
---

import PipsPagerDefaultScreenshot from '/img/controls/pipspager/pipspager-default.png';
import PipsPagerCarouselScreenshot from '/img/controls/pipspager/pipspager-carousel.png';
import PipsPagerLargeCollectionScreenshot from '/img/controls/pipspager/pipspager-large-collection.png';
import PipsPagerCustomColorsScreenshot from '/img/controls/pipspager/pipspager-custom-colors.png';
import PipsPagerCustomButtonsScreenshot from '/img/controls/pipspager/pipspager-custom-buttons.png';
import PipsPagerPillTemplateScreenshot from '/img/controls/pipspager/pipspager-pill-template.png';

# PipsPager

`PipsPager` 是一个页面指示器控件，用一排可交互的圆点（pips）表示分页集合中的各个页面。用户可以点击某个圆点，或使用可选的“上一页/下一页”导航按钮来切换选中页面。当页面数量超过 `MaxVisiblePips` 时，圆点会自动滚动，以保持当前选中的圆点始终可见。

`PipsPager` 通常与 `Carousel` 或 `CarouselPage` 配合使用，并通过 `SelectedPageIndex` 进行绑定。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `NumberOfPages` | `int` | `0` | 圆点所表示的总页数。 |
| `SelectedPageIndex` | `int` | `0` | 当前选中页面的从零开始索引。支持双向绑定。其值会被限制在 `[0, NumberOfPages - 1]` 范围内。 |
| `MaxVisiblePips` | `int` | `5` | 一次最多可见的圆点数量。当 `NumberOfPages` 超过该值时，圆点会自动滚动。最小值为 `1`。 |
| `Orientation` | `Orientation` | `Horizontal` | 圆点的布局方向，可为 `Horizontal` 或 `Vertical`。 |
| `IsPreviousButtonVisible` | `bool` | `true` | 显示或隐藏“上一页”导航按钮。 |
| `IsNextButtonVisible` | `bool` | `true` | 显示或隐藏“下一页”导航按钮。 |
| `PreviousButtonTheme` | `ControlTheme?` | `null` | 应用于“上一页”导航按钮的自定义主题。 |
| `NextButtonTheme` | `ControlTheme?` | `null` | 应用于“下一页”导航按钮的自定义主题。 |

## 伪类

| 伪类 | 条件 |
| ------------ | --------- |
| `:first-page` | `SelectedPageIndex` 为 `0`。 |
| `:last-page` | `SelectedPageIndex` 为 `NumberOfPages - 1` 且 `NumberOfPages > 0`。 |
| `:horizontal` | `Orientation` 为 `Horizontal`。 |
| `:vertical` | `Orientation` 为 `Vertical`。 |

## 事件

| 事件 | 参数类型 | 说明 |
| ----- | --------- | ----------- |
| `SelectedIndexChanged` | `PipsPagerSelectedIndexChangedEventArgs` | 当选中页面发生变化时引发。提供 `OldIndex` 和 `NewIndex`。 |

## 键盘导航

- 左箭头 / 上箭头：移动到上一页。
- 右箭头 / 下箭头：移动到下一页。
- Home：跳转到第一页（索引 `0`）。
- End：跳转到最后一页（`NumberOfPages - 1`）。

## 样式资源键

你可以在 `PipsPager` 或其祖先元素上覆盖这些资源键，以自定义圆点指示器颜色：

| 资源键 | 说明 |
| ------------ | ----------- |
| `PipsPagerSelectionIndicatorForeground` | 默认圆点颜色。 |
| `PipsPagerSelectionIndicatorForegroundSelected` | 选中圆点的颜色。 |
| `PipsPagerSelectionIndicatorForegroundPointerOver` | 指针悬停时的圆点颜色。 |
| `PipsPagerSelectionIndicatorForegroundPressed` | 按下时的圆点颜色。 |

## 示例

### 基础 PipsPager

```xml
<PipsPager NumberOfPages="5" />
```

<Image light={PipsPagerDefaultScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 在代码中使用 PipsPager

```csharp
var pager = new PipsPager
{
    NumberOfPages = 5,
    MaxVisiblePips = 5
};
```

### 垂直方向

```xml
<PipsPager NumberOfPages="5" Orientation="Vertical" />
```

### 不显示导航按钮

```xml
<PipsPager NumberOfPages="10"
           MaxVisiblePips="5"
           IsPreviousButtonVisible="False"
           IsNextButtonVisible="False" />
```

### 与 Carousel 进行双向绑定

将 `SelectedPageIndex` 绑定到 `Carousel.SelectedIndex`，以实现同步导航：

```xml
<Grid RowDefinitions="*,Auto">
    <Carousel Name="GalleryCarousel"
              SelectedIndex="{Binding #GalleryPager.SelectedPageIndex, Mode=TwoWay}">
        <Carousel.Items>
            <Border Background="#E3F2FD" CornerRadius="8">
                <TextBlock Text="Page 1" FontSize="30"
                           VerticalAlignment="Center" HorizontalAlignment="Center" />
            </Border>
            <Border Background="#C8E6C9" CornerRadius="8">
                <TextBlock Text="Page 2" FontSize="30"
                           VerticalAlignment="Center" HorizontalAlignment="Center" />
            </Border>
            <Border Background="#FFE0B2" CornerRadius="8">
                <TextBlock Text="Page 3" FontSize="30"
                           VerticalAlignment="Center" HorizontalAlignment="Center" />
            </Border>
        </Carousel.Items>
    </Carousel>

    <PipsPager Name="GalleryPager"
               Grid.Row="1"
               NumberOfPages="3"
               HorizontalAlignment="Center"
               Margin="0,12,0,0" />
</Grid>
```

<Image light={PipsPagerCarouselScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 与 CarouselPage 进行双向绑定

```xml
<Grid RowDefinitions="*,Auto">
    <CarouselPage Name="DemoCarousel"
                  SelectedIndex="{Binding #Pager.SelectedPageIndex, Mode=TwoWay}">
        <ContentPage Header="Welcome">
            <TextBlock Text="Welcome" FontSize="28"
                       HorizontalAlignment="Center" VerticalAlignment="Center" />
        </ContentPage>
        <ContentPage Header="Features">
            <TextBlock Text="Features" FontSize="28"
                       HorizontalAlignment="Center" VerticalAlignment="Center" />
        </ContentPage>
        <ContentPage Header="Get Started">
            <TextBlock Text="Get Started" FontSize="28"
                       HorizontalAlignment="Center" VerticalAlignment="Center" />
        </ContentPage>
    </CarouselPage>

    <PipsPager Name="Pager"
               Grid.Row="1"
               NumberOfPages="3"
               HorizontalAlignment="Center"
               Margin="0,12,0,0" />
</Grid>
```

### 大集合与自动滚动圆点

当 `NumberOfPages` 超过 `MaxVisiblePips` 时，圆点条会自动滚动，以保持当前选中的圆点可见：

```xml
<PipsPager NumberOfPages="50"
           MaxVisiblePips="7"
           SelectedPageIndex="25" />
```

<Image light={PipsPagerLargeCollectionScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 响应选择变化

```csharp
pager.SelectedIndexChanged += (sender, e) =>
{
    Console.WriteLine($"Changed from page {e.OldIndex} to {e.NewIndex}");
};
```

### 自定义圆点颜色

在 `PipsPager` 上覆盖指示器资源键，以更改圆点颜色：

```xml
<PipsPager NumberOfPages="5" MaxVisiblePips="5">
    <PipsPager.Resources>
        <SolidColorBrush x:Key="PipsPagerSelectionIndicatorForeground" Color="Orange" />
        <SolidColorBrush x:Key="PipsPagerSelectionIndicatorForegroundSelected" Color="Blue" />
        <SolidColorBrush x:Key="PipsPagerSelectionIndicatorForegroundPointerOver" Color="Gold" />
    </PipsPager.Resources>
</PipsPager>
```

<Image light={PipsPagerCustomColorsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 自定义按钮主题

使用 `PreviousButtonTheme` 和 `NextButtonTheme` 将默认的箭头按钮替换为自定义主题按钮：

```xml
<PipsPager NumberOfPages="5" MaxVisiblePips="5">
    <PipsPager.Resources>
        <ControlTheme x:Key="PrevTheme" TargetType="Button">
            <Setter Property="Content" Value="Prev" />
            <Setter Property="Background" Value="LightGray" />
            <Setter Property="Foreground" Value="Black" />
            <Setter Property="Padding" Value="8,2" />
            <Setter Property="Margin" Value="0,0,8,0" />
            <Setter Property="Template">
                <ControlTemplate>
                    <Border Background="{TemplateBinding Background}" CornerRadius="4">
                        <ContentPresenter Content="{TemplateBinding Content}"
                                          Margin="{TemplateBinding Padding}" />
                    </Border>
                </ControlTemplate>
            </Setter>
        </ControlTheme>
        <ControlTheme x:Key="NextTheme" TargetType="Button">
            <Setter Property="Content" Value="Next" />
            <Setter Property="Background" Value="LightGray" />
            <Setter Property="Foreground" Value="Black" />
            <Setter Property="Padding" Value="8,2" />
            <Setter Property="Margin" Value="8,0,0,0" />
            <Setter Property="Template">
                <ControlTemplate>
                    <Border Background="{TemplateBinding Background}" CornerRadius="4">
                        <ContentPresenter Content="{TemplateBinding Content}"
                                          Margin="{TemplateBinding Padding}" />
                    </Border>
                </ControlTemplate>
            </Setter>
        </ControlTheme>
    </PipsPager.Resources>
    <PipsPager.PreviousButtonTheme>
        <StaticResource ResourceKey="PrevTheme" />
    </PipsPager.PreviousButtonTheme>
    <PipsPager.NextButtonTheme>
        <StaticResource ResourceKey="NextTheme" />
    </PipsPager.NextButtonTheme>
</PipsPager>
```

<Image light={PipsPagerCustomButtonsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 自定义圆点模板（胶囊形指示器）

使用针对内部 `ListBoxItem` 的样式选择器来替换默认圆点形状。这个示例会将选中的圆点从圆形变成水平胶囊形：

```xml
<PipsPager NumberOfPages="5"
           IsPreviousButtonVisible="False"
           IsNextButtonVisible="False">
    <PipsPager.Styles>
        <Style Selector="PipsPager /template/ ListBox ListBoxItem">
            <Setter Property="Width" Value="24" />
            <Setter Property="Height" Value="24" />
            <Setter Property="Padding" Value="0" />
            <Setter Property="Margin" Value="2,0" />
            <Setter Property="MinWidth" Value="0" />
            <Setter Property="MinHeight" Value="0" />
            <Setter Property="VerticalAlignment" Value="Center" />
            <Setter Property="Template">
                <ControlTemplate>
                    <Grid Background="Transparent">
                        <Border Name="Pip"
                                Width="8" Height="8" CornerRadius="4"
                                HorizontalAlignment="Center" VerticalAlignment="Center"
                                Background="#C0C0C0">
                            <Border.Transitions>
                                <Transitions>
                                    <DoubleTransition Property="Width" Duration="0:0:0.2" Easing="CubicEaseOut" />
                                    <DoubleTransition Property="Height" Duration="0:0:0.2" Easing="CubicEaseOut" />
                                    <CornerRadiusTransition Property="CornerRadius" Duration="0:0:0.2" Easing="CubicEaseOut" />
                                    <BrushTransition Property="Background" Duration="0:0:0.2" />
                                </Transitions>
                            </Border.Transitions>
                        </Border>
                    </Grid>
                </ControlTemplate>
            </Setter>
        </Style>
        <Style Selector="PipsPager /template/ ListBox ListBoxItem:pointerover /template/ Border#Pip">
            <Setter Property="Width" Value="10" />
            <Setter Property="Height" Value="10" />
            <Setter Property="CornerRadius" Value="5" />
            <Setter Property="Background" Value="#909090" />
        </Style>
        <Style Selector="PipsPager /template/ ListBox ListBoxItem:selected /template/ Border#Pip">
            <Setter Property="Width" Value="24" />
            <Setter Property="Height" Value="8" />
            <Setter Property="CornerRadius" Value="4" />
            <Setter Property="Background" Value="#FF6B35" />
        </Style>
        <Style Selector="PipsPager /template/ ListBox ListBoxItem:selected:pointerover /template/ Border#Pip">
            <Setter Property="Width" Value="24" />
            <Setter Property="Height" Value="8" />
            <Setter Property="CornerRadius" Value="4" />
            <Setter Property="Background" Value="#E85A2A" />
        </Style>
    </PipsPager.Styles>
</PipsPager>
```

<Image light={PipsPagerPillTemplateScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 以编程方式控制

```csharp
// 设置页数
pager.NumberOfPages = 20;

// 跳转到指定页面
pager.SelectedPageIndex = 10;

// 限制可见圆点数量
pager.MaxVisiblePips = 7;

// 切换为垂直方向
pager.Orientation = Orientation.Vertical;

// 隐藏导航按钮
pager.IsPreviousButtonVisible = false;
pager.IsNextButtonVisible = false;
```

## 另请参阅

- [API 参考](/api/avalonia/controls/pipspager)
- [源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/PipsPager/PipsPager.cs)
