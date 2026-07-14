---
id: visual-and-logical-trees
title: 视觉树与逻辑树
description: 理解用于布局、渲染和事件处理的逻辑树与视觉树结构。
doc-type: explanation
---

Avalonia 将控件组织为两棵并行的树结构：逻辑树和视觉树。理解这两棵树，对于资源查找、事件路由、样式系统以及自定义控件开发都非常重要。

## 逻辑树

逻辑树表示的是你在 XAML 中定义出来的 UI 结构。它只包含你显式声明的控件，而不包含控件模板内部展开出来的结构。

例如，下面这段 XAML：

```xml
<Window>
    <StackPanel>
        <TextBlock Text="Name:" />
        <TextBox Text="{Binding Name}" />
        <Button Content="Save" />
    </StackPanel>
</Window>
```

会生成如下逻辑树：

```text
Window
  └─ StackPanel
       ├─ TextBlock
       ├─ TextBox
       └─ Button
```

逻辑树主要用于：
- **资源查找**：`StaticResource` 和 `DynamicResource` 会沿逻辑树向上查找。
- **数据上下文继承**：`DataContext` 会沿逻辑树向下传播。
- **属性继承**：例如 `FontSize`、`Foreground` 和 `FlowDirection` 这类可继承属性会沿逻辑树向下流动。
- **命名元素查找**：`x:Name` 引用会在逻辑树作用域内解析。

### 遍历逻辑树

```csharp
// 获取逻辑父级
var parent = myControl.Parent;

// 获取逻辑子级
foreach (var child in myPanel.Children)
{
    // 处理子元素
}

// 查找特定类型的祖先
var window = myControl.FindLogicalAncestorOfType<Window>();

// 获取所有逻辑后代
var allTextBlocks = myPanel.GetLogicalDescendants().OfType<TextBlock>();
```

## 视觉树

视觉树包含所有参与渲染的可视元素，包括控件模板内部的各个组成部分。逻辑树中的一个 `Button`，在视觉树中会展开为它的 `ContentPresenter`、`Border` 以及其他模板元素。

上面那个同样的 `Button`，在视觉树中可能会像这样展开：

```text
Button
  └─ ContentPresenter
       └─ Border
            └─ ContentPresenter
                 └─ TextBlock ("Save")
```

视觉树主要用于：
- **渲染**：渲染器会遍历视觉树来绘制 UI。
- **命中测试**：指针事件会借助视觉树判断鼠标或触点当前位于哪个元素之上。
- **布局**：测量和排列过程都会遍历视觉树。
- **事件隧道与冒泡**：路由事件会沿视觉树向下隧道、向上冒泡。

### 遍历视觉树

```csharp
// 获取视觉父级
var parent = myControl.GetVisualParent();

// 获取视觉子级
var childCount = myControl.VisualChildren.Count;
var firstChild = myControl.VisualChildren[0];

// 查找视觉祖先
var scrollViewer = myControl.FindAncestorOfType<ScrollViewer>(includeSelf: false);

// 查找满足条件的视觉祖先
var enabledPanel = myControl.FindAncestorOfType<StackPanel>(
    includeSelf: false,
    predicate: panel => panel.IsEnabled);

// 查找满足条件的视觉后代
var visibleTextBox = myPanel.FindDescendantOfType<TextBox>(
    includeSelf: false,
    predicate: tb => tb.IsVisible);

// 获取所有视觉后代
var allVisuals = myControl.GetVisualDescendants();
```

## 两棵树的区别

| 方面 | 逻辑树 | 视觉树 |
|---|---|---|
| 包含内容 | 你在 XAML 中声明的控件 | 所有可视元素，包括模板内部结构 |
| 资源查找 | 是 | 否 |
| 数据上下文继承 | 是 | 否 |
| 属性继承 | 是 | 否 |
| 事件路由 | 否 | 是（隧道与冒泡） |
| 命中测试 | 否 | 是 |
| 布局 | 部分参与 | 是 |

## 何时使用哪一棵树

**以下情况适合使用逻辑树：**
- 查找资源或数据上下文
- 查找命名元素
- 从某个控件向上查找其逻辑父级
- 处理 `ItemsControl` 的子项（这些项位于逻辑树中）

**以下情况适合使用视觉树：**
- 查找控件模板内部的组成部分
- 遍历实际渲染出来的元素层级
- 实现命中测试
- 在元素之间转换坐标（`TranslatePoint`）

## 运行时查看树结构

你可以使用 Avalonia DevTools（在调试构建中按 F12）以交互方式查看这两棵树。DevTools 中的 Logical Tree 和 Visual Tree 标签页会展示完整层级以及各项属性。

```csharp
// 输出逻辑树，便于调试
static void PrintLogicalTree(StyledElement element, int indent = 0)
{
    Debug.WriteLine($"{new string(' ', indent * 2)}{element.GetType().Name}");
    if (element is ILogical logical)
    {
        foreach (var child in logical.LogicalChildren)
        {
            if (child is StyledElement styledChild)
                PrintLogicalTree(styledChild, indent + 1);
        }
    }
}
```

## 模板部件与视觉树

当你创建控件模板时，模板中的元素会成为视觉树的一部分，但不会进入逻辑树。若要在自定义控件中访问这些模板部件，可以重写 `OnApplyTemplate`：

```csharp
protected override void OnApplyTemplate(TemplateAppliedEventArgs e)
{
    base.OnApplyTemplate(e);

    // 从模板中查找命名元素
    var border = e.NameScope.Find<Border>("PART_Border");
    var textBlock = e.NameScope.Find<TextBlock>("PART_Text");
}
```

按照惯例，模板部件通常会以 `PART_` 作为前缀命名，以便和逻辑子元素区分开来。

## 覆盖层

Avalonia 会在每个窗口中，位于普通控件内容之上维护几个特殊层。这些层用于处理装饰器、自定义覆盖内容以及弹出内容：

| 层 | 用途 | 访问方式 |
|---|---|---|
| `AdornerLayer` | 焦点指示器、拖拽装饰和附加到控件上的视觉装饰。 | `AdornerLayer.GetAdornerLayer(visual)` |
| `OverlayLayer` | 位于普通控件之上、弹出层之下的自定义覆盖内容。 | `OverlayLayer.GetOverlayLayer(visual)` |
| Popup layer | 承载弹出内容（菜单、工具提示、下拉框等）的内部层，由框架管理。 | 由弹出系统内部管理 |

### 添加自定义覆盖内容

你可以使用 `OverlayLayer` 来显示悬浮在普通视觉树之上的内容，例如加载指示器、浮动工具栏或自定义通知面板：

```csharp
var overlay = OverlayLayer.GetOverlayLayer(myControl);
if (overlay is not null)
{
    var panel = new Border
    {
        Background = Brushes.Black,
        Opacity = 0.5,
        Child = new TextBlock
        {
            Text = "Loading...",
            Foreground = Brushes.White,
            HorizontalAlignment = HorizontalAlignment.Center,
            VerticalAlignment = VerticalAlignment.Center,
        }
    };

    overlay.Children.Add(panel);

    // 使用完后移除
    overlay.Children.Remove(panel);
}
```

覆盖层内容会显示在窗口中的所有普通控件之上，但仍位于弹出窗口、菜单和工具提示之下。

## 另请参阅

- [UI composition](/docs/fundamentals/ui-composition): How controls compose into a UI.
- [Control trees](/docs/custom-controls/control-trees): Visual and logical trees in custom control development.
- [Events overview](/docs/events): How events route through the visual tree.
- [Templated controls](/docs/custom-controls/templated-controls): Building controls with templates.
