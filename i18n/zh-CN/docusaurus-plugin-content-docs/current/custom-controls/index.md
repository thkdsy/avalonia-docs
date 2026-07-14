---
id: index
title: 创建自定义控件
sidebar_position: 1
description: 概览 Avalonia 中构建自定义控件的几种方式，从用户控件到模板控件。
doc-type: overview
---

## 自定义控件

自定义控件使用 Avalonia 图形系统自行绘制，可以调用绘制形状、线条、填充、文本等基础方法。你还可以定义自己的属性、事件和伪类。

Avalonia 的一些内置控件本身就是这样实现的，例如文本块控件（`TextBlock` 类）和图像控件（`Image` 类）。

## 自定义控件的类型

如果你想创建自己的控件，Avalonia 中主要有三大类控件可供选择。首先要做的，就是挑选最适合你使用场景的控件类别。

### 用户控件

`UserControl` 是编写控件最简单的方式。这类控件最适合用于某个应用专属的“视图”或“页面”。`UserControl` 的编写方式与 `Window` 基本相同：从模板新建一个 `UserControl`，然后向其中添加控件即可。

### 模板控件

`TemplatedControl` 最适合用于可以在多个应用之间共享的通用控件。它们属于无外观控件，也就是说可以针对不同主题和应用重新定义样式。Avalonia 中绝大多数标准控件都属于这一类。

:::info
在 WPF/UWP 中，你通常会继承 `Control` 类来创建新的模板控件；但在 Avalonia 中，应继承 `TemplatedControl`。
:::

:::info
如果你想在单独文件中为 `TemplatedControl` 提供 `Style`，请记得通过 `StyleInclude` 将该文件引入到应用中。
:::

### 基础控件

基础控件是用户界面的基础构件——它们通过重写 `Visual.Render` 方法，使用几何绘制来自行呈现。像 `TextBlock` 和 `Image` 这样的控件就属于这一类。

:::info
在 WPF/UWP 中，你通常会继承 `FrameworkElement` 来创建新的基础控件；但在 Avalonia 中，应继承 `Control`。
:::

## 创建高级自定义控件

下面展示了 `Border` 控件如何定义它的 `Background` 属性：

`AvaloniaProperty.Register` 方法还支持若干其他参数：

* `defaultValue`：为属性提供默认值。这里只应传递值类型和不可变类型，因为如果传入引用类型，就会导致所有注册该属性的实例共享同一个对象。
* `inherits`：指定该属性的默认值是否应从父控件继承。
* `defaultBindingMode`：该属性的默认绑定模式。可以设置为 `OneWay`、`TwoWay`、`OneTime` 或 `OneWayToSource`。
* `validate`：一个类型为 `Func<TOwner, TValue, TValue>` 的验证/强制修正函数。该函数会接收正在设置属性的类实例及传入的值，并返回修正后的值，或者在值无效时抛出异常。

:::info
样式属性类似于其他 XAML 框架中的 `DependencyProperty`。
:::

:::info
属性及其后备 `AvaloniaProperty` 字段的命名约定非常重要。字段名始终应为属性名后追加 `Property` 后缀。
:::

### 在另一个类上使用 `StyledProperty`

有时你想给控件添加的属性，已经存在于另一个控件上，`Background` 就是一个很好的例子。若要注册在另一个控件上定义的属性，可以调用 `StyledProperty.AddOwner`：

```csharp
public static readonly StyledProperty<IBrush> BackgroundProperty =
    Border.BackgroundProperty.AddOwner<Panel>();

public Brush Background
{
    get { return GetValue(BackgroundProperty); }
    set { SetValue(BackgroundProperty, value); }
}
```

:::note
与 WPF/UWP 不同，属性必须先注册到某个类上，否则就无法在该类对象上设置这个属性。不过这一点未来可能会有所变化。
:::

### 只读属性

要创建只读属性，可以使用 `AvaloniaProperty.RegisterDirect` 方法。下面是 `Visual` 注册只读 `Bounds` 属性的方式：

```csharp
public static readonly DirectProperty<Visual, Rect> BoundsProperty =
    AvaloniaProperty.RegisterDirect<Visual, Rect>(
        nameof(Bounds),
        o => o.Bounds);

private Rect _bounds;

public Rect Bounds
{
    get { return _bounds; }
    private set { SetAndRaise(BoundsProperty, ref _bounds, value); }
}
```

正如你所见，只读属性会作为对象上的字段存储。在注册属性时，需要传入一个 getter，它用于通过 `GetValue` 访问属性值；而 `SetAndRaise` 则用于在属性变化时通知监听者。

### 附加属性

[附加属性](/docs/custom-controls/attached-properties) 的定义方式与样式属性几乎完全相同，不同之处在于它们通过 `RegisterAttached` 方法注册，并且访问器定义为静态方法。

下面展示了 `Grid` 如何定义 `Grid.Column` 附加属性：

```csharp
public static readonly AttachedProperty<int> ColumnProperty =
    AvaloniaProperty.RegisterAttached<Grid, Control, int>("Column");

public static int GetColumn(Control element)
{
    return element.GetValue(ColumnProperty);
}

public static void SetColumn(Control element, int value)
{
    element.SetValue(ColumnProperty, value);
}
```

### 直接 `AvaloniaProperty` 注册

顾名思义，`RegisterDirect` 并不仅仅用于注册只读属性。你也可以向 `RegisterDirect` 传入一个 setter，把标准 C# 属性暴露为 Avalonia 属性。

通过 `AvaloniaProperty.Register` 注册的 `StyledProperty` 会维护一个带优先级的值与绑定列表，以支持样式系统正常工作。但对于很多属性来说，这种机制其实有些过度，例如 `ItemsControl.Items` —— 这个属性永远不会被样式化，因此样式属性带来的额外开销没有必要。

下面是 `ItemsControl.Items` 的注册方式：

```csharp
public static readonly DirectProperty<ItemsControl, IEnumerable> ItemsProperty =
    AvaloniaProperty.RegisterDirect<ItemsControl, IEnumerable>(
        nameof(Items),
        o => o.Items,
        (o, v) => o.Items = v);

private IEnumerable _items = new AvaloniaList<object>();

public IEnumerable Items
{
    get { return _items; }
    set { SetAndRaise(ItemsProperty, ref _items, value); }
}
```

直接属性是样式属性的轻量版本，它支持以下能力：

* `AvaloniaObject.GetValue`
* 对于非只读属性可使用 `AvaloniaObject.SetValue`
* `PropertyChanged`
* 绑定（仅支持 `LocalValue` 优先级）
* `GetObservable`
* `AddOwner`
* 元数据

它们不支持以下能力：

* 验证/强制修正（虽然这可以在属性 setter 中手动完成）
* 覆盖默认值
* 继承值

### 在另一个类上使用 `DirectProperty`

就像你可以对样式属性调用 `AddOwner` 一样，你也可以为直接属性添加新的 owner。由于直接属性引用的是控件上的字段，因此你还必须为该属性添加一个字段：

```csharp
public static readonly DirectProperty<MyControl, IEnumerable> ItemsProperty =
    ItemsControl.ItemsProperty.AddOwner<MyControl>(
        o => o.Items,
        (o, v) => o.Items = v);

private IEnumerable _items = new AvaloniaList<object>();

public IEnumerable Items
{
    get { return _items; }
    set { SetAndRaise(ItemsProperty, ref _items, value); }
}
```

### 何时使用直接属性或样式属性

通常情况下，你应优先将属性声明为样式属性。不过，直接属性也有各自的优缺点：

优点：

* 不会为每个实例额外分配属性对象
* 属性 getter 是标准 C# getter
* 属性 setter 是标准 C# setter，并且会触发事件
* 你可以添加[数据验证](/docs/app-development/data-validation)支持

缺点：

* 不能从父控件继承值
* 不能利用 Avalonia 的样式系统
* 属性值以字段形式存在，因此无论是否被设置，都会占用存储空间

因此，在以下需求下适合使用直接属性：

* 属性不需要参与样式
* 属性通常或始终都会有值

### DataValidation 支持

若要让某个属性显示验证错误信息，请在注册时设置 `enableDataValidation: true`。基础 `Control` 类会自动把验证错误上报给 `DataValidationErrors`，因此不需要额外重写任何逻辑。

**启用 DataValidation 的属性示例**

```csharp
public static readonly DirectProperty<MyControl, int> ValueProperty =
    AvaloniaProperty.RegisterDirect<MyControl, int>(
        nameof(Value),
        o => o.Value,
        (o, v) => o.Value = v,
        enableDataValidation: true);
```

这对 `DirectProperty` 和 `StyledProperty` 注册方式都有效。当绑定提供验证错误时，控件会自动将其显示出来。

如果你想[复用另一个类的直接属性](#using-a-directproperty-on-another-class)，同样也可以启用数据验证。这种情况下请使用 `AddOwnerWithDataValidation`。

**示例：`TextBox.TextProperty` 复用了 `TextBlock.TextProperty`，并增加了验证支持**

```csharp
public static readonly DirectProperty<TextBox, string?> TextProperty =
    TextBlock.TextProperty.AddOwnerWithDataValidation<TextBox>(
        o => o.Text,
        (o, v) => o.Text = v,
        defaultBindingMode: BindingMode.TwoWay,
        enableDataValidation: true);
```

在 Avalonia 12 中，只要将 `enableDataValidation` 设置为 `true`，数据验证就会被自动处理。框架会检测来自 `INotifyDataErrorInfo` 的验证错误，并自动显示出来，无需你在控件中做额外重写。

## 另请参阅

- [选择自定义控件类型](/docs/custom-controls/choosing-a-custom-control-type)
- [定义属性](/docs/custom-controls/defining-properties)
- [定义事件](/docs/custom-controls/defining-events)
- [模板控件](/docs/custom-controls/templated-controls)
- [绘制自定义控件](/docs/custom-controls/drawing-custom-controls)
