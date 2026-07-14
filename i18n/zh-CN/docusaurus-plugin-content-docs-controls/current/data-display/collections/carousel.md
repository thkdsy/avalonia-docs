---
id: carousel
title: Carousel
description: Avalonia 中 Carousel 控件的参考文档，介绍如何逐页显示项目、配置页面过渡动画、实现导航以及使用数据绑定。
doc-type: reference
---

import CarouselScreenshot from '/img/reference/controls/carousel/carousel.gif';

`Carousel` 拥有一个项目集合，并按顺序将每个项目显示为单独的一页，使其填满整个控件。你可以用它来构建幻灯片、引导流程，或任何需要让用户逐页浏览内容的界面。

## 常用属性

以下是最常使用的属性：

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `PageTransition` | `IPageTransition?` | `null` | 选中项目变化时播放的过渡动画。内置选项包括 `PageSlide`、`CrossFade`、`Rotate3DTransition` 和 `CompositePageTransition`。 |
| `IsSwipeEnabled` | `bool` | `false` | 启用滑动和指针拖拽手势，以在页面之间导航。 |
| `ViewportFraction` | `double` | `1.0` | 每一页占视口的比例。小于 `1.0` 的值会露出相邻页面的一部分，例如 `0.8` 可预览相邻页，`0.33` 大致可同时显示三个项目。 |
| `IsSwiping` | `bool` | `false` | 只读。滑动手势进行中时为 `true`。 |
| `WrapSelection` | `bool` | `false` | 为 `true` 时，`Next()` 会从最后一项循环到第一项，`Previous()` 会从第一项循环到最后一项。 |
| `SelectedIndex` | `int` | `-1` | 当前显示项目的从零开始索引。 |
| `SelectedItem` | `object?` | `null` | 绑定集合中当前显示的项目。 |
| `ItemsSource` | `IEnumerable?` | `null` | 作为数据源使用的绑定集合。 |
| `ItemTemplate` | `IDataTemplate?` | `null` | 应用于每个项目的 `DataTemplate`，用于控制项目外观。 |
| `ItemsPanel` | `ITemplate<Panel?>` | `VirtualizingCarouselPanel` | 用于排列项目的容器面板。有关自定义项目面板的详细信息，请参阅 [ItemsControl](/controls/data-display/collections/itemscontrol)。 |
| `AutoScrollToSelectedItem` | `bool` | `true` | 自动滚动，使选中项目进入视图。 |

## 示例

此示例在项目集合中放置了三张图片，并提供按钮向前或向后切换显示内容。这些按钮在 C# 代码后置中绑定了点击事件处理程序。

```xml title='XAML'
<Panel>
    <Carousel Name="slides">
        <Carousel.PageTransition>
            <CompositePageTransition>
                <PageSlide Duration="0:00:01.500" Orientation="Horizontal" />
            </CompositePageTransition>
        </Carousel.PageTransition>
        <Carousel.Items>
            <Image Source="avares://AvaloniaControls/Assets/pipes.jpg" />
            <Image Source="avares://AvaloniaControls/Assets/controls.jpg" />
            <Image Source="avares://AvaloniaControls/Assets/vault.jpg" />
        </Carousel.Items>
    </Carousel>
    <Panel Margin="20">
        <Button Background="White" Click="Previous">&lt;</Button>
        <Button Background="White" Click="Next"
                HorizontalAlignment="Right">&gt;</Button>
    </Panel>
</Panel>
```

```csharp title='C#'
using Avalonia.Controls;
using Avalonia.Interactivity;

namespace AvaloniaControls.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        public void Next(object source, RoutedEventArgs args)
        {
            slides.Next();
        }

        public void Previous(object source, RoutedEventArgs args)
        {
            slides.Previous();
        }
    }
}
```

<Image light={CarouselScreenshot} alt="Carousel 控件在多张幻灯片之间循环切换" position="center" maxWidth={400} cornerRadius="true"/>

## 绑定到集合

使用 `ItemsSource` 可将 `Carousel` 绑定到数据集合，并提供自定义 `DataTemplate`：

```xml title='XAML'
<Carousel ItemsSource="{Binding Slides}" SelectedIndex="{Binding CurrentSlide}">
    <Carousel.PageTransition>
        <CrossFade Duration="0:00:00.300" />
    </Carousel.PageTransition>
    <Carousel.ItemTemplate>
        <DataTemplate>
            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                <TextBlock Text="{Binding Title}" FontSize="24" FontWeight="Bold" />
                <TextBlock Text="{Binding Description}" TextWrapping="Wrap" />
            </StackPanel>
        </DataTemplate>
    </Carousel.ItemTemplate>
</Carousel>
```

视图模型公开了集合和当前索引：

```csharp title='C#'
public class SlidesViewModel : ViewModelBase
{
    public ObservableCollection<Slide> Slides { get; } = new()
    {
        new Slide("Welcome",   "Get started with Avalonia."),
        new Slide("Features",  "Cross-platform, high-performance UI."),
        new Slide("Community", "Join the Avalonia community today."),
    };

    private int _currentSlide;
    public int CurrentSlide
    {
        get => _currentSlide;
        set => this.RaiseAndSetIfChanged(ref _currentSlide, value);
    }
}
```

由于 `SelectedIndex` 默认采用双向绑定，你可以通过修改 `CurrentSlide` 从视图模型推进轮播，也可以让控件在用户通过按钮导航时自动更新该属性。

## 页面过渡

你可以通过为 `PageTransition` 属性指定过渡对象，来设置项目之间切换时播放的动画。Avalonia 内置了多种过渡效果：

| 过渡效果 | 说明 |
|---|---|
| `PageSlide` | 从指定方向滑入内容。可将 `Orientation` 设置为 `Horizontal`（默认）或 `Vertical`。 |
| `CrossFade` | 通过透明度动画淡出当前项目并淡入新项目。 |
| `Rotate3DTransition` | 在 3D 空间中旋转当前项目和进入的项目，支持水平与垂直轴。 |
| `CompositePageTransition` | 组合多个过渡效果并让它们同时运行。 |

### `PageSlide` 示例

```xml title='XAML'
<Carousel.PageTransition>
    <PageSlide Duration="0:00:00.500" Orientation="Horizontal" />
</Carousel.PageTransition>
```

### `CrossFade` 示例

```xml title='XAML'
<Carousel.PageTransition>
    <CrossFade Duration="0:00:00.300" />
</Carousel.PageTransition>
```

### 组合过渡示例

你可以使用 `CompositePageTransition` 将多个过渡叠加在一起：

```xml title='XAML'
<Carousel.PageTransition>
    <CompositePageTransition>
        <CrossFade Duration="0:00:00.500" />
        <PageSlide Duration="0:00:00.500" Orientation="Horizontal" />
    </CompositePageTransition>
</Carousel.PageTransition>
```

### 禁用过渡

若要在切换项目时完全不使用动画，可将 `PageTransition` 设置为 `{x:Null}`：

```xml title='XAML'
<Carousel PageTransition="{x:Null}" />
```

有关页面过渡的完整指南（包括如何创建自定义过渡），请参阅 [设置页面过渡](/docs/graphics-animation/page-transitions)。

## 导航

你可以通过多种方式切换当前显示的项目：

| 方式 | 说明 |
|---|---|
| `Next()` | 前进到集合中的下一个项目。 |
| `Previous()` | 返回到上一个项目。 |
| `SelectedIndex` | 设置或绑定要显示项目的从零开始索引。 |
| `SelectedItem` | 直接设置或绑定项目对象。 |

### 使用按钮导航（代码后置）

最简单的方式是在按钮点击事件处理程序中调用 `Next()` 和 `Previous()`，如上方的[示例](#示例)所示。

### 使用数据绑定导航

将 `SelectedIndex` 绑定到视图模型中的属性，这样你就可以从应用逻辑中控制导航：

```xml title='XAML'
<Carousel ItemsSource="{Binding Pages}" SelectedIndex="{Binding PageIndex}" />
```

```csharp title='C#'
public int PageIndex
{
    get => _pageIndex;
    set => this.RaiseAndSetIfChanged(ref _pageIndex, value);
}

public void GoToNext()
{
    if (PageIndex < Pages.Count - 1)
        PageIndex++;
}

public void GoToPrevious()
{
    if (PageIndex > 0)
        PageIndex--;
}
```

## 滑动手势

通过设置 `IsSwipeEnabled` 来启用滑动和指针拖拽导航：

```xml title='XAML'
<Carousel IsSwipeEnabled="True">
    <!-- items -->
</Carousel>
```

启用后，用户可以在页面之间拖动，并获得可视反馈。控件也支持快速甩动手势，需要达到一定滑动速度阈值后才会完成切换。手势进行期间，`IsSwiping` 属性会为 `true`。

## 循环选择

启用循环后，在最后一项上调用 `Next()` 会跳回第一项，而在第一项上调用 `Previous()` 会跳到最后一项：

```xml title='XAML'
<Carousel WrapSelection="True">
    <!-- items -->
</Carousel>
```

## 视口占比

将 `ViewportFraction` 设为小于 `1.0` 的值，即可在选中页旁边露出相邻页面：

```xml title='XAML'
<Carousel ViewportFraction="0.8">
    <!-- items -->
</Carousel>
```

值为 `1.0`（默认）时，只显示单个完整页面。像 `0.8` 这样的值会产生“预览”效果，让相邻页面的边缘可见；`0.33` 则大致可同时容纳三个项目进入视图。

## 键盘导航

`Carousel` 在获得焦点时支持键盘导航：

| 按键 | 操作 |
|---|---|
| 左箭头 / 上箭头 | 移动到上一个项目。 |
| 右箭头 / 下箭头 | 移动到下一个项目。 |
| Home | 跳转到第一个项目。 |
| End | 跳转到最后一个项目。 |

## 另请参阅

- [PipsPager](/controls/layout/containers/pipspager) 点状分页指示器
- [CarouselPage](/controls/navigation/carouselpage) 基于页面的轮播导航
- [设置页面过渡](/docs/graphics-animation/page-transitions)
- [TransitioningContentControl](/controls/data-display/transitioningcontentcontrol)
- [ItemsControl](/controls/data-display/collections/itemscontrol)
- [ListBox](/controls/data-display/collections/listbox)
- [Carousel API 参考](/api/avalonia/controls/carousel)
- [`Carousel.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Carousel.cs)
