---
id: transitioningcontentcontrol
title: TransitioningContentControl
description: 一个在内容变化时播放过渡动画的内容控件，支持交叉淡入淡出、滑动和组合页面过渡效果。
doc-type: reference
---

import TransitioningContentControlFadeScreenshot from '/img/controls/transitioningcontentcontrol/transitioningcontentcontrol-fade.webp';
import TransitioningContentControlSlideScreenshot from '/img/controls/transitioningcontentcontrol/transitioningcontentcontrol-slide.webp';

[`TransitioningContentControl`](/api/avalonia/controls/transitioningcontentcontrol) 一次只显示一个内容项，并在该内容发生变化时播放带动画的页面过渡效果。它继承自 [`ContentControl`](/controls/data-display/contentcontrol)，因此凡是可以使用普通内容控件的地方，你通常也可以使用它。

一个常见的使用场景是构建图片幻灯片，但 `TransitioningContentControl` 同样非常适合在导航场景中切换不同视图。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
|---|---|
| `Content` | 要在控件中显示的内容。 |
| `ContentTemplate` | 用于显示内容的 `DataTemplate`。 |
| `PageTransition` | 用于在内容变化时执行动画的页面过渡效果。所应用的主题会提供默认过渡。将此属性设置为 `{x:Null}` 可完全禁用过渡动画。 |
| `IsTransitionReversed` | 设置为 `true` 时，过渡动画会反向播放（例如从滑入变为滑出）。 |

## 内置页面过渡效果

Avalonia 提供了多种可应用到 `TransitioningContentControl` 的页面过渡效果：

| 过渡效果 | 说明 |
|---|---|
| `CrossFade` | 在淡出旧内容的同时淡入新内容。 |
| `PageSlide` | 让内容从指定方向滑入。支持 `Horizontal` 和 `Vertical` 方向。 |
| `CompositePageTransition` | 组合多个过渡效果，使其同时运行。 |

你也可以通过实现 `IPageTransition` 来创建自定义过渡效果。完整说明请参阅 [Setting page transitions](../../docs/graphics-animation/page-transitions)。

## 示例

### 默认过渡效果（交叉淡入淡出）

在这个示例中，视图模型中包含一个图片集合。下面的 XAML 使用默认页面过渡效果，在绑定的 `SelectedImage` 属性变化时为图片切换添加动画：

```xml
<TransitioningContentControl Content="{Binding SelectedImage}">
    <TransitioningContentControl.ContentTemplate>
        <DataTemplate DataType="Bitmap">
            <Image Source="{Binding}" />
        </DataTemplate>
    </TransitioningContentControl.ContentTemplate>
</TransitioningContentControl>
```

<Image light={TransitioningContentControlFadeScreenshot} alt="TransitioningContentControl with default cross-fade transition" position="center" maxWidth={400} cornerRadius="true"/>

### 水平滑动过渡

你可以通过设置 `PageTransition` 来替换默认过渡效果。这里使用 `PageSlide` 让图片进行水平滑动切换：

```xml
<TransitioningContentControl Content="{Binding SelectedImage}">
    <TransitioningContentControl.PageTransition>
        <PageSlide Orientation="Horizontal" Duration="0:00:00.500" />
    </TransitioningContentControl.PageTransition>
    <TransitioningContentControl.ContentTemplate>
        <DataTemplate DataType="Bitmap">
            <Image Source="{Binding}" />
        </DataTemplate>
    </TransitioningContentControl.ContentTemplate>
</TransitioningContentControl>
```

<Image light={TransitioningContentControlSlideScreenshot} alt="TransitioningContentControl with horizontal page-slide transition" position="center" maxWidth={400} cornerRadius="true"/>

### 自定义时长的交叉淡入淡出

```xml
<TransitioningContentControl Content="{Binding CurrentView}">
    <TransitioningContentControl.PageTransition>
        <CrossFade Duration="0:00:00.300" />
    </TransitioningContentControl.PageTransition>
</TransitioningContentControl>
```

### 禁用过渡效果

如果你希望内容立即切换而不播放动画，请将 `PageTransition` 设置为 null：

```xml
<TransitioningContentControl Content="{Binding CurrentView}"
                             PageTransition="{x:Null}" />
```

## 结合数据模板切换视图

`TransitioningContentControl` 常用于为视图之间的导航切换添加动画。你可以将 `Content` 绑定到视图模型属性，并为每种视图模型类型提供一个 `DataTemplate`。当该属性发生变化时，控件会自动解析正确的模板，并以过渡动画切换到新视图。

```xml
<TransitioningContentControl Content="{Binding CurrentPage}">
    <TransitioningContentControl.PageTransition>
        <PageSlide Duration="0:00:00.300" Orientation="Horizontal" />
    </TransitioningContentControl.PageTransition>
    <TransitioningContentControl.DataTemplates>
        <DataTemplate DataType="vm:HomeViewModel">
            <views:HomeView />
        </DataTemplate>
        <DataTemplate DataType="vm:SettingsViewModel">
            <views:SettingsView />
        </DataTemplate>
    </TransitioningContentControl.DataTemplates>
</TransitioningContentControl>
```

如需完整的操作示例，请参阅 [How to set up basic navigation](../../docs/how-to/navigation-how-to)。

## 另请参阅

- [ContentControl](/controls/data-display/contentcontrol)
- [Setting page transitions](../../docs/graphics-animation/page-transitions)
- [How to set up basic navigation](../../docs/how-to/navigation-how-to)
- [Carousel](/controls/data-display/collections/carousel)
- [`TransitioningContentControl` API reference](https://reference.avaloniaui.net/api/Avalonia.ReactiveUI/TransitioningContentControl/)
- [`TransitioningContentControl` source code](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TransitioningContentControl.cs)
