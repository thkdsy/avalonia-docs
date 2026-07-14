---
id: app-performance-issues
title: 应用性能问题
---

你的应用运行太慢了吗？通过在开发过程中考虑几个关键事项，可以显著提升 Avalonia 应用的性能。本文讨论了你可以采取哪些步骤来优化 Avalonia 应用的性能。

## 使用 CompiledBindings

在 Avalonia 中提升性能的最有效方式之一，是在应用中使用 [`CompiledBindings`](/docs/data-binding/compiled-bindings)。编译绑定通过在编译时编译绑定路径来实现更快的数据绑定，从而减少运行时反射的开销。 

## 为数据展示选择合适的控件

当你需要在 `DataGrid` 中显示大量数据，或者在具有许多节点的 `TreeView` 中显示大量数据时，建议使用 `TreeDataGrid` 控件。`TreeDataGrid` 是从零开始构建的，性能优于普通的 `DataGrid`。它支持虚拟化，尤其适合需要虚拟化树的场景，因为它支持层级数据模板。

如果你不需要编辑功能，请避免使用 `DataGrid` 控件。一般认为它在性能方面并不是最优的控件。

:::caution
截至 2025 年 10 月，[`TreeDataGrid`](/controls/data-display/structured-data/treedatagrid) 作为 Avalonia Pro 的一部分进行维护。目前它仍然是大规模数据集的推荐选项。
:::

## 虚拟化

在处理大量数据时，启用虚拟化可以提升 Avalonia 应用的性能。虚拟化意味着控件中只有可见项会被渲染，这在需要显示大量项目时能显著提高性能。

### TreeDataGrid

`TreeDataGrid` 支持虚拟化，并且能够有效处理带有复杂单元格的数千行数据。

:::caution
截至 2025 年 10 月，[`TreeDataGrid`](/controls/data-display/structured-data/treedatagrid) 作为 Avalonia Pro 的一部分进行维护。目前它仍然是大规模数据集的推荐选项。
:::

## 优化你的可视树结构

性能常常会受到过深且复杂的布局结构影响。应尽量保持你的 XAML 标记尽可能简洁、扁平。渲染屏幕上的 UI 元素时，每个元素都会触发两次“布局过程”（一次测量过程，随后一次排列过程）。

这个布局过程计算量很大：某个项拥有的子元素越多，需要进行的计算就越多。因此，尽量降低 Avalonia 中可视树的复杂度，可以显著提升应用性能。

## 尽量减少使用 run 来设置文本属性

建议尽量减少在 `TextBlock` 中使用 `Run`，因为这可能导致更消耗资源的操作。如果你正在使用 `Run` 来定义文本属性，考虑直接在 `TextBlock` 上设置这些属性。这样做有助于提升应用性能。

## 使用 StreamGeometries 而不是 PathGeometries

在处理 Avalonia 中的几何图形时，`StreamGeometry` 比 `PathGeometry` 更高效。`StreamGeometry` 专门针对处理大量 `PathGeometry` 对象进行了优化，占用更少内存，并提供更优的性能。因此，在有选择的情况下，建议使用 `StreamGeometry` 替代 `PathGeometry`，以提升应用性能。

## 使用更小的图像尺寸

当你的应用需要显示较小的图像或缩略图时，生成并使用缩小尺寸的图像版本会更有益。默认情况下，Avalonia 会按图像原始完整尺寸加载并解码，这在你加载大图并在 `ItemsControl` 之类的控件中将其缩放到缩略图大小时，可能会导致性能瓶颈。

## 解决你的绑定错误 

绑定错误是 Avalonia 应用中常见的性能问题来源。每次发生绑定错误时，应用在尝试解析绑定并将错误记录到跟踪日志时，性能都会下降。很自然，绑定错误越多，对性能的影响就越大。 

导致绑定错误的一个重要原因是在 `DataTemplates` 中使用 `RelativeSource` 绑定，因为在 `DataTemplate` 完成初始化之前，绑定通常无法正确解析。建议完全避免使用 `RelativeSource.FindAncestor`。一种更高效的方法是定义一个附加属性，并利用属性继承将值向下传递到可视树中，而不是去查找可视树。

## 异步加载数据

性能问题、UI 卡顿以及应用无响应，通常都源于数据加载方式。为避免压垮 UI 线程，请确保以异步方式加载数据。