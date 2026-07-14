---
id: layout
title: 布局
description: Avalonia 布局系统如何使用面板和边界框来测量并排列控件。
doc-type: explanation
---

import LayoutZonesDiagram from '/img/concepts/ui-concepts/layout/layout-zones.png';

Avalonia 布局系统通过“测量”和“排列”两个阶段来确定控件的位置和大小。本页将介绍该系统的工作原理、可用的面板类型以及边界框模型。

## 面板

Avalonia 提供了一组继承自 `Panel` 的元素。这些 `Panel` 元素可以实现多种复杂布局。例如，使用 `StackPanel` 可以轻松完成堆叠布局，而使用 [`Canvas`](/api/avalonia/controls/canvas) 则可以实现更复杂、更自由的布局。

下表总结了可用的 `Panel` 控件：

| 名称            | 说明                                                                                                                                                                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Panel`         | 将所有子元素布局为填满 `Panel` 的边界。 |
| `Canvas`        | 定义一个区域，你可以在其中使用相对于 Canvas 区域的坐标显式定位子元素。 |
| `DockPanel`     | 定义一个区域，可让你相对于彼此按水平或垂直方向排列子元素。 |
| [`Grid`](/api/avalonia/controls/grid)          | 定义由行和列组成的灵活网格区域。 |
| `RelativePanel` | 相对于其他元素或面板自身来排列子元素。 |
| `StackPanel`    | 将子元素排列成一条单线，可设置为水平或垂直方向。 |
| `WrapPanel`     | 从左到右顺序放置子元素，在到达容器边界时自动换行。后续内容会根据 `Orientation` 属性的值，按从上到下或从右到左的方向继续排列。 |

在 WPF 中，`Panel` 是一个抽象类；如果想让多个控件填满可用空间，通常需要使用一个没有行列定义的 `Grid`。而在 Avalonia 中，`Panel` 本身就是可直接使用的控件，它具有与“无行列定义的 `Grid`”相同的布局行为，但运行时开销更小。

## 元素边界框

在理解 Avalonia 布局时，掌握包围所有元素的边界框非常重要。布局系统中的每个 `Control` 都可以被视为一个被放入布局槽位中的矩形。`Bounds` 属性返回元素在布局中所分配到的边界。这个矩形的大小由多种因素共同决定，包括可用屏幕空间、各种约束尺寸、布局相关属性（如 margin 和 padding），以及父级 `Panel` 元素自身的行为。布局系统会根据这些数据计算特定 `Panel` 下所有子元素的位置。还要注意，定义在父元素上的尺寸特征（例如 `Border`）也会影响它的子元素。

## 布局系统

从最简单的角度看，布局是一个递归系统，它最终会让元素被确定尺寸、确定位置并绘制出来。更具体地说，布局描述的是测量和排列 `Panel` 元素 `Children` 集合成员的过程。布局是一个计算密集型过程；`Children` 集合越大，需要进行的计算就越多。复杂度还会受到拥有该集合的 `Panel` 所定义布局行为的影响。像 `Canvas` 这样相对简单的 `Panel`，性能通常会明显好于像 `Grid` 这样更复杂的 `Panel`。

每当子控件的位置发生变化时，都有可能触发布局系统重新执行一次布局过程。因此，理解哪些事件会触发布局系统非常重要，因为不必要的触发会降低应用性能。下面描述了布局系统被调用时发生的过程。

1. 子 `Control` 会先测量其核心属性，从而开始布局过程。
2. 评估 `Control` 上定义的尺寸属性，例如 `Width`、`Height` 和 `Margin`。
3. 应用 `Panel` 特有的布局逻辑，例如 `Dock` 方向或堆叠 `Orientation`。
4. 在所有子元素测量完成后对内容进行排列。
5. 将 `Children` 集合绘制到屏幕上。
6. 如果又有新的 `Children` 被加入集合，则会再次触发这一过程。

接下来的章节会更详细地说明这一过程及其触发方式。

## 测量和排列子元素

布局系统会对 `Children` 集合中的每个成员执行两次处理：一次测量阶段（measure pass），一次排列阶段（arrange pass）。每个子 `Panel` 都通过自身的 `MeasureOverride` 和 `ArrangeOverride` 方法来实现特定的布局行为。

在测量阶段，`Children` 集合中的每个成员都会被评估。该过程从调用 `Measure` 方法开始。这个方法由父级 `Panel` 元素的实现内部调用，因此不需要显式调用它也会发生布局。

首先，会评估 `Visual` 的原生尺寸相关属性，例如 `Clip` 和 `IsVisible`。如果 `IsVisible` 为 `false`，该控件会被完全排除在布局之外：布局系统会将它的 `DesiredSize` 设为零，并跳过其子树。这也意味着渲染器不会绘制该控件。关于将 `IsVisible` 和 `Opacity` 作为隐藏策略进行比较，可参阅 [IsVisible vs Opacity](/docs/graphics-animation/effects#isvisible-vs-opacity)。这些原生属性的评估结果会生成一个约束，并传递给 `MeasureCore`。

接下来，会处理影响该约束值的框架属性。这些属性通常描述底层 `Control` 的尺寸特征，例如 `Height`、`Width` 和 `Margin`。这些属性中的每一个都可能改变显示该元素所需的空间。随后会以该约束为参数调用 `MeasureOverride`。

由于 `Bounds` 是一个计算得出的值，因此要注意：在布局系统执行各种操作时，它可能会出现多次或渐进式的变化。例如，布局系统可能正在为子元素计算所需测量空间，或处理中父元素施加的约束等。

测量阶段的最终目标，是让子元素确定自己的 `DesiredSize`，这一过程发生在 `MeasureCore` 调用期间。`Measure` 会保存该 `DesiredSize`，以便在后续的排列阶段使用。

排列阶段从调用 `Arrange` 方法开始。在排列阶段，父级 `Panel` 元素会生成一个矩形，用于表示子元素的边界。该值随后会被传递给 `ArrangeCore` 方法进行处理。

`ArrangeCore` 方法会评估子元素的 `DesiredSize`，并处理可能影响元素最终渲染大小的额外 margin。`ArrangeCore` 会生成一个排列尺寸，并将其作为参数传递给 `Panel` 的 `ArrangeOverride` 方法。`ArrangeOverride` 则负责生成子元素的 `finalSize`。最后，`ArrangeCore` 会对 margin、alignment 等偏移属性再做一次最终评估，并将子元素放入它的布局槽位中。子元素不一定要填满整个已分配空间，事实上它经常不会填满。随后控制权返回给父级 `Panel`，布局过程完成。

## 布局区域

<Image light={LayoutZonesDiagram} maxWidth="400" alignment="center" alt="一个由四个相互重叠的矩形组成的示意图，表示 UI 窗口的布局区域。" />

## 叠加层

除了常规布局系统外，Avalonia 还提供了叠加层，可在窗口中的普通控件内容之上进行渲染。当你需要把某些内容显示在所有普通内容之上时，它非常有用，例如加载指示器、浮动工具栏或通知面板。

可以使用 `OverlayLayer.GetOverlayLayer(visual)` 获取指定 visual 对应的叠加层表面。添加到 `OverlayLayer` 的内容会显示在所有普通控件之上，但仍位于弹出窗口、菜单和工具提示之下。

有关详细信息和代码示例，请参阅 [叠加层](/docs/fundamentals/visual-and-logical-trees#overlay-layers)。

## 另请参阅

- [控件定位](/docs/layout/positioning-controls)：对齐、边距和定位。
- [响应式布局](/docs/layout/responsive-layouts)：使用容器查询和可回流面板适应不同尺寸的布局。
- [叠加层](/docs/fundamentals/visual-and-logical-trees#overlay-layers)：在普通控件上方添加自定义叠加内容。
