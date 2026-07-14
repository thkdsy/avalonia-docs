---
id: choosing-a-custom-control-type
title: 选择自定义控件类型
description: 比较用户控件、模板控件和自绘控件，选择最适合你场景的方案。
doc-type: explanation
---

Avalonia 提供了多种创建自定义控件的方法，以满足应用程序的特定需求。了解不同类型的自定义控件，有助于你根据实际要求选择最合适的方案。在 Avalonia 中，三种常见的自定义控件类型分别是 [`UserControl`](/api/avalonia/controls/usercontrol)、无外观控件（lookless control）和自绘控件。

## UserControl

`UserControl` 是 Avalonia 中用于创建自定义控件的一种高级方式。它允许你通过组合现有控件并使用 XAML 定义布局来构建控件。`UserControl` 充当一个容器，将多个控件封装在一起，并提供统一的用户界面。

:::info
通常，`UserControl` 用于表示应用程序中的某个专用视图，例如“用户详情视图”，而不是作为通用用户界面元素使用。
:::

创建 `UserControl` 通常包含以下步骤：

1. **定义 XAML**：创建新的 `UserControl` XAML 文件，通过放置控件、设置属性和应用样式来定义控件的布局与外观。

2. **代码后置**：你也可以按需添加代码后置逻辑，用于处理事件、调整行为，或为 `UserControl` 提供额外功能。

3. **复用与自定义**：`UserControl` 可以在应用中轻松复用和定制。它尤其适合将一组特定控件和行为封装成可复用组件或“视图”。


<GitHubSampleLink title="Custom Control" link="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/CustomControl"/>


## 模板控件（无外观控件）

模板控件（也称为“无外观控件”）为在 Avalonia 中创建自定义控件提供了更高级、可定制性更强的方式。模板控件会将控件的行为与逻辑和其视觉外观分离，使应用开发者可以独立地为控件定义样式和模板。

使用模板控件时，你需要在代码类中定义控件的行为和属性，而视觉表现则通过 XAML 中定义的控件模板来指定。这种分离让应用开发者无需修改底层行为，就能自定义控件的外观与交互体验。

:::info
模板控件通常用于与业务逻辑无关的通用界面元素，并且它们往往需要支持不同主题或视觉样式。Avalonia 提供的大多数[内置控件](/controls)都是模板控件。
:::

创建模板控件通常包含以下步骤：

1. **定义控件类**：创建一个继承自 `TemplatedControl` 的新类。该类负责定义控件的行为、属性和事件。

2. **控件模板**：在 XAML 中创建 [`ControlTheme`](/docs/styling/control-themes)，用于指定控件的视觉外观和结构。控件模板定义了控件由哪些部分组成，以及这些部分应如何设置样式。

3. **样式与模板定制**：应用开发者可以通过修改控件模板或应用样式来定制控件外观，从而在整个应用中保持一致统一的视觉设计。

模板控件具有更高的灵活性和复用性，非常适合需要适配不同视觉主题或各种用户偏好的场景。

## 自绘控件

自绘控件在 Avalonia 中提供了最高级别的定制能力。使用自绘控件时，你可以完全控制控件视觉元素的渲染过程，从而创建独特且复杂的视觉表现。

:::info
自绘控件通常适用于主要表示非交互式图形元素、且不需要主题化的场景。
:::

要创建自绘控件，需要重写控件的 `Render` 方法，并使用 `DrawingContext` 等底层绘图 API 来定义控件外观。这种方式能够细粒度地控制控件视觉表现中的每一个像素。

创建自绘控件通常包含以下步骤：

1. **定义控件类**：创建一个继承自 `Control` 的新类。该类将定义控件的行为和渲染逻辑。

2. **重写 Render 方法**：在控件类中重写 `Render` 方法，并使用 `DrawingContext` 绘制控件内容。

## 另请参阅

- [自定义控件类](/docs/custom-controls/custom-control-class)
- [模板控件](/docs/custom-controls/templated-controls)
- [绘制自定义控件](/docs/custom-controls/drawing-custom-controls)
- [定义属性](/docs/custom-controls/defining-properties)
