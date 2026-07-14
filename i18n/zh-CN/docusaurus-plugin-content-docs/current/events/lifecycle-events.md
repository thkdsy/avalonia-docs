---
id: lifecycle-events
title: 生命周期事件
description: Avalonia 中控件初始化、附加到视觉树以及卸载过程中的事件。
doc-type: reference
---

Avalonia 控件在创建、附加到视觉树以及被移除的过程中，会触发多个事件。理解这些事件的顺序和用途，对于初始化控件、加载数据以及清理资源都非常重要。

## 生命周期事件顺序

当控件被创建并添加到视觉树中时，事件会按以下顺序触发：

| 顺序 | 事件 / 方法 | 定义于 | 说明 |
|---|---|---|---|
| 1 | `Initialized` | `StyledElement` | 来自 XAML 的所有属性值都已设置完毕，但控件此时还不属于视觉树。 |
| 2 | `AttachedToVisualTree` | `Visual` | 控件已被加入到一个有根的视觉树中，但布局尚未发生。 |
| 3 | `Loaded` | `Control` | 控件已完全附加，并且已可交互。该事件会在视觉树附加流程完成后触发。 |

当控件被移除时：

| 顺序 | 事件 / 方法 | 定义于 | 说明 |
|---|---|---|---|
| 1 | `Unloaded` | `Control` | 控件即将从视觉树中移除。 |
| 2 | `DetachedFromVisualTree` | `Visual` | 控件已经从视觉树中移除。 |

## Initialized

当 XAML 加载器完成对标记中定义的所有属性赋值后，就会触发 `Initialized` 事件。此时控件的属性值已经设置好，但控件可能还没有加入视觉树。

```csharp
public class MyControl : Control
{
    protected override void OnInitialized()
    {
        base.OnInitialized();
        // 来自 XAML 的属性已经设置完毕
        // 但视觉树可能还不可用
    }
}
```

也可以在外部订阅：

```csharp
myControl.Initialized += (sender, e) =>
{
    // 控件已初始化
};
```

**适用场景**：初始化依赖 XAML 属性值、但不依赖视觉树的内部状态。

## AttachedToVisualTree / DetachedFromVisualTree

当控件被添加到或移除出一个有根的视觉树（即根节点为 `TopLevel` 的视觉树）时，就会触发这些事件。

```csharp
public class MyControl : Control
{
    protected override void OnAttachedToVisualTree(VisualTreeAttachmentEventArgs e)
    {
        base.OnAttachedToVisualTree(e);
        // e.RootVisual 是视觉树的根节点
        // 可以开始监听外部服务、定时器等
    }

    protected override void OnDetachedFromVisualTree(VisualTreeAttachmentEventArgs e)
    {
        base.OnDetachedFromVisualTree(e);
        // 清理外部订阅、定时器等资源
    }
}
```

`VisualTreeAttachmentEventArgs` 提供以下信息：

| 属性 | 类型 | 说明 |
|---|---|---|
| `RootVisual` | `Visual` | 控件所附加到的那棵树的根视觉对象。 |
| `AttachmentPoint` | `Visual` | 控件直接附加到或脱离的视觉对象。 |
| `PresentationSource` | `IPresentationSource` | 承载该视觉树的呈现源。 |

**适用场景**：订阅或取消订阅那些只应在控件可见时保持活动的外部服务、平台 API 或事件。

## Loaded / Unloaded

当控件附加到视觉树并完成相关初始化后，会触发 `Loaded` 事件；而当控件被移除时，则会触发 `Unloaded` 事件。

```csharp
public class MyControl : Control
{
    protected override void OnLoaded(RoutedEventArgs e)
    {
        base.OnLoaded(e);
        // 控件已完全就绪
        // 布局已完成，绑定也已激活
    }

    protected override void OnUnloaded(RoutedEventArgs e)
    {
        base.OnUnloaded(e);
        // 清理资源
    }
}
```

也可以在 XAML/代码中订阅：

```csharp
myControl.Loaded += (sender, e) =>
{
    // 控件已加载并且可用
};
```

**适用场景**：执行那些要求控件已完全就绪且视觉树已激活的操作，例如启动动画、测量布局或拉取数据。

### Loaded 与 AttachedToVisualTree 的区别

这两个事件都表明控件已成为视觉树的一部分。关键区别在于：

- `AttachedToVisualTree` 会在控件进入视觉树时立刻触发。它是 `Visual` 上的普通 CLR 事件。
- `Loaded` 会在附加流程完全完成后触发。它是 `Control` 上的 `RoutedEvent`。

对于大多数场景，`Loaded` 才是更合适的选择。只有当你需要访问 `Root` 引用，或者正在处理非 `Control` 类型的视觉对象时，才更适合使用 `AttachedToVisualTree`。

## DataContextChanged

当 `StyledElement` 上的 `DataContext` 属性发生变化时，就会触发 `DataContextChanged` 事件：

```csharp
myControl.DataContextChanged += (sender, e) =>
{
    var newContext = ((Control)sender!).DataContext;
    // 对新的数据上下文做出响应
};
```

该事件会在以下情况下触发：
- 直接在控件上设置了 `DataContext`。
- 因为父级 `DataContext` 改变，导致继承来的 `DataContext` 发生变化。
- 控件移动到了视觉树中的另一个位置，而该位置继承的是不同的 `DataContext`。

## 典型初始化模式

### 在视图中加载数据

```csharp
public partial class CustomerView : UserControl
{
    public CustomerView()
    {
        InitializeComponent();
    }

    protected override void OnLoaded(RoutedEventArgs e)
    {
        base.OnLoaded(e);

        if (DataContext is CustomerViewModel vm)
        {
            vm.LoadCustomersCommand.Execute(null);
        }
    }
}
```

### 管理订阅

```csharp
public class StatusMonitor : Control
{
    private IDisposable? _subscription;

    protected override void OnAttachedToVisualTree(VisualTreeAttachmentEventArgs e)
    {
        base.OnAttachedToVisualTree(e);
        _subscription = StatusService.StatusChanged.Subscribe(OnStatusChanged);
    }

    protected override void OnDetachedFromVisualTree(VisualTreeAttachmentEventArgs e)
    {
        _subscription?.Dispose();
        _subscription = null;
        base.OnDetachedFromVisualTree(e);
    }

    private void OnStatusChanged(string status)
    {
        // 更新控件
    }
}
```

## 另请参阅

- [Events Overview](/docs/events)：路由事件系统的工作方式。
- [Application Lifetimes](/docs/fundamentals/application-lifetimes)：应用级生命周期事件。
- [UI Composition](/docs/fundamentals/ui-composition)：控件如何在视觉树中进行组合。
