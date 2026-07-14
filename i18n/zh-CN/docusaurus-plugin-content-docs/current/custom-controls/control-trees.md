---
id: control-trees
title: 控件树
description: 了解 Avalonia 如何将控件组织为逻辑树和可视树，以支持布局和渲染。
doc-type: explanation
---

import ControlTreesLogicalScreenshot from '/img/concepts/ui-concepts/controls/control-trees-logical.png';
import ControlTreesVisualScreenshot from '/img/concepts/ui-concepts/controls/control-trees-visual.png';
import ControlTreesEventScreenshot from '/img/concepts/ui-concepts/controls/control-trees-events.png';

Avalonia 会根据应用中的 XAML 文件创建控件树，从而实现界面渲染并管理应用功能。

## 逻辑树

逻辑控件树表示应用中的控件（包括主窗口）在 XAML 中定义时的层级结构。例如，窗口中一个控件（按钮）位于另一个控件（堆栈面板）内部时，其逻辑树就会呈现出如下三层结构：

<Image light={ControlTreesLogicalScreenshot} alt="Logical tree structure of an Avalonia window" position="center" maxWidth={400} cornerRadius="true"/>

当应用运行时，你可以打开 _Avalonia Dev Tools_ 窗口（按 <kbd>F12</kbd>）。其中的 **Logical Tree** 选项卡会显示逻辑树。

## 可视树

可视控件树包含了 Avalonia 实际运行时所涉及的一切。它会显示控件上设置的所有属性，以及 Avalonia 为呈现界面和管理应用功能而额外添加的所有组成部分。

<Image light={ControlTreesVisualScreenshot} alt="Visual tree structure showing template-expanded controls" position="center" maxWidth={400} cornerRadius="true"/>

你可以在 _Avalonia Dev Tools_ 窗口的 **Visual Tree** 选项卡中查看可视树。

## 事件

Avalonia 在管理应用功能时，一个关键部分就是事件的生成与传播。**Events** 选项卡会记录你在运行中的应用里移动、点击或进行其他交互时事件的来源和传播过程。

<Image light={ControlTreesEventScreenshot} alt="Event routing through the control tree" position="center" maxWidth={400} cornerRadius="true"/>

## 逻辑树与可视树对比

这两棵树承担着不同职责，并包含不同层级的细节。

| 方面 | 逻辑树 | 可视树 |
|---|---|---|
| 包含内容 | 在 XAML 中声明的控件 | 所有已渲染元素，包括模板部件 |
| 模板展开 | 否（模板仍视为单个节点） | 是（模板会展开为内部部件） |
| 主要用途 | DataContext 继承、资源查找 | 渲染、命中测试、布局 |
| 导航 API | `LogicalExtensions.GetLogicalParent()` | `VisualExtensions.GetVisualParent()` |

## 在代码中遍历树

Avalonia 提供了扩展方法，可用于以编程方式遍历这两棵树。

```csharp
// 向上遍历逻辑树
var parent = myControl.GetLogicalParent();

// 向上遍历可视树
var visualParent = myControl.GetVisualParent();

// 在逻辑树中查找特定祖先
var window = myControl.FindLogicalAncestorOfType<Window>();

// 在可视树中查找特定祖先
var border = myControl.FindAncestorOfType<Border>();

// 查找满足条件的祖先或后代
var enabledPanel = myControl.FindAncestorOfType<StackPanel>(
    predicate: p => p.IsEnabled);
var visibleTextBox = myControl.FindDescendantOfType<TextBox>(
    predicate: tb => tb.IsVisible);

// 获取所有逻辑子节点
var children = myControl.GetLogicalChildren();
```

## 自定义控件中的树遍历

在构建自定义控件时，你经常需要响应控件被添加到树中或从树中移除的时机。可以重写以下方法来挂接树生命周期事件：

- `OnAttachedToLogicalTree` / `OnDetachedFromLogicalTree`：用于设置和清理数据绑定、订阅或继承属性。
- `OnAttachedToVisualTree` / `OnDetachedFromVisualTree`：用于执行与渲染相关的初始化和清理，例如获取平台资源。

```csharp
protected override void OnAttachedToVisualTree(VisualTreeAttachmentEventArgs e)
{
    base.OnAttachedToVisualTree(e);
    // 控件现在已成为可视树的一部分，可以参与渲染
}

protected override void OnDetachedFromVisualTree(VisualTreeAttachmentEventArgs e)
{
    base.OnDetachedFromVisualTree(e);
    // 清理渲染资源
}
```

## 另请参阅

- [可视树与逻辑树](/docs/fundamentals/visual-and-logical-trees)：基础概念。
- [模板控件](/docs/custom-controls/templated-controls)：模板如何展开到可视树中。
