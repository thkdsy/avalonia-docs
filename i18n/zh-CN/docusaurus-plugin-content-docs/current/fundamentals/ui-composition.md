---
id: ui-composition
title: UI 组合
description: 使用窗口、内置控件、用户控件和自定义控件来组合界面布局。
doc-type: explanation
---

import CompositionBasicLayoutDiagram from '/img/concepts/core-concepts/ui-composition/composition-basic-layout.png';
import CompositionTreesDiagram from '/img/concepts/core-concepts/ui-composition/composition-trees.png';
import CompositionUserControlsDiagram from '/img/concepts/core-concepts/ui-composition/composition-usercontrol.png';
import CompositionCollectionControlsDiagram from '/img/concepts/core-concepts/ui-composition/composition-collection-controls.png';

UI 组合是指你用来创建应用程序所需布局的过程。它让你可以通过组织多个组件来构建复杂视图。它的优点包括：

* _封装_ - 通过把每个组件的 XAML 和代码限制在它真正需要的范围内，来降低复杂度，使代码更易理解和维护。
* _复用_ - 让应用程序中重复出现的部分保持一致的外观与行为。

Avalonia 支持 UI 组合，因此你可以构建应用程序所需的各种布局和功能结构。

当你使用 Avalonia 构建应用程序时，可以选择多种不同类型的组件：

* Windows
* 内置控件
* 用户控件
* 自定义控件
* 模板控件

## 窗口与内置控件

在 Avalonia 中，窗口是布局的一个基本单元（针对具备窗口系统的平台）。

Avalonia 提供了大量内置控件，能够覆盖大多数 UI 需求。  

<Image light={CompositionBasicLayoutDiagram} alt="Diagram showing a single control and a multi-control layout inside a window" position="center" maxWidth={400} cornerRadius="true"/>

当你刚开始接触 Avalonia 时，可能会先在窗口的内容区域中放置一个单独的内置控件（见上图左侧）。这是最简单的一种 UI 组合形式：窗口具有应用标题，并且通常带有一些窗口状态控制项（取决于目标平台）。而内置控件则负责接收用户输入，或者以特定布局和样式显示输出内容。

稍微复杂一点的应用，可能会需要使用某个内置布局控件，在窗口内容区中排列多个内置控件（见上图右侧）。

:::info
如果想查看 Avalonia 提供的全部内置控件，请参阅 [Controls reference](/controls)。
:::

## 逻辑树与视觉树

无论你采用什么样的控件排列方式，Avalonia 都会把它们之间的关系表示为一棵树结构，其中最外层的控件作为根节点。例如，前面的 UI 组合就可以表示为这里展示的树结构：

<Image light={CompositionTreesDiagram} alt="Diagram showing the logical control tree for a window with nested controls" position="center" maxWidth={400} cornerRadius="true"/>

这就是 **逻辑控件树**。它按照控件在 XAML 中定义的层级关系来表示应用程序中的控件（包括主窗口）。Avalonia 中有许多系统都会处理逻辑控件树，以及与它配套的 **视觉控件树**。

:::info
关于控件树概念的更多说明，请参阅 [Control trees](/docs/custom-controls/control-trees)。
:::

## 用户控件

用户控件是 Avalonia 中进行 UI 组合的核心手段之一。

<Image light={CompositionUserControlsDiagram} alt="Diagram showing user controls used as page views and reusable components" position="center" maxWidth={400} cornerRadius="true"/>

你可以把用户控件添加到主窗口的内容区域中，把它作为一个“页面视图”（见上图左侧）。这样你就能够实现更复杂的多页面应用；每个页面的布局和功能都分别放在自己的用户控件（XAML 和代码）文件中。  

用户控件的另一种常见用途，是作为一个组件控件（见上图右侧）。你一开始可能只是为了降低窗口或页面视图的复杂度而这样做；但之后你也很可能会把这个组件复用到其他页面中。

## 教程

:::info
有关 `DataTemplates` 的教程，请参阅 [Avalonia.Samples](https://github.com/AvaloniaUI/Avalonia.Samples/tree/main?tab=readme-ov-file#%EF%B8%8F-datatemplate-samples)。  
:::

## 集合控件

UI 组合的另一种常见场景，是你需要展示一个项目集合。

<Image light={CompositionCollectionControlsDiagram} alt="Diagram showing a collection control with a data template rendering items" position="center" maxWidth={400} cornerRadius="true"/>

在这种场景中，通常会使用某种内置的重复控件，并将其绑定到一个集合上，再配合数据模板来呈现集合中的各个项目。

:::info
有关数据模板和集合控件的更多信息，请参阅 [DataTemplate samples](https://github.com/AvaloniaUI/Avalonia.Samples/tree/main?tab=readme-ov-file#%EF%B8%8F-datatemplate-samples)。
:::

## 自定义控件

如果在少数情况下，你确实找不到任何 Avalonia 内置控件能够满足应用的 UI 需求，那么你可以从零开始“自己造”一个自定义控件。这样你就可以定义自己的属性、事件和方法；但与此同时，你也需要自己实现该控件的外观绘制逻辑。

:::info
如果想学习如何实现自定义控件，请参阅 [Custom controls](/docs/custom-controls)。
:::

## 模板控件

模板控件使用 Avalonia 的 **样式** 系统来创建可复用控件，其外观由控件模板定义。这使你能够在不改变控件行为的前提下，调整它的视觉结构。

:::info
如果想了解 Avalonia **样式** 系统背后的更多概念，请参阅 [Styles](/docs/styling/styles)。
:::

## 另请参阅

- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)
- [Code-behind](/docs/fundamentals/code-behind)
- [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)
- [Controls reference](/controls)
