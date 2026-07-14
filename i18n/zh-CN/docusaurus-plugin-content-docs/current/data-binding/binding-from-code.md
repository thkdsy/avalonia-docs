---
id: binding-from-code
title: 如何在代码中进行绑定
doc-type: how-to
description: 通过 C# 代码而不是 XAML 来创建和管理数据绑定。
---

## 在代码中创建编译绑定

[`CompiledBinding.Create`](/api/avalonia/data/compiledbinding#create-method) 方法允许你使用 LINQ 表达式创建类型安全的绑定。表达式会在编译期验证，因此如果属性名写错，会直接产生编译错误，而不是在运行时悄悄失效。

该方法接收两个泛型参数：[`DataContext`](/docs/data-binding/data-context) 的类型，以及要选择的属性类型。随后再传入一个 lambda 表达式，用来指定要绑定的属性。

例如，如果某个控件的 [`DataContext`](/docs/data-binding/data-context) 是 `MyViewModel` 的实例，而你想从中选择一个 `string Title { get; set; }` 属性进行绑定，可以这样写：

```csharp
var binding = CompiledBinding.Create<MyViewModel, string>(x => x.Title);
```

然后把它绑定到控件上：

```csharp
textBlock.Bind(TextBlock.TextProperty, binding);
```

[`CompiledBinding.Create`](/api/avalonia/data/compiledbinding#create-method) 还支持多个可选参数，用于控制绑定行为。

下面的示例展示了如何从一个显式指定的 `viewModel` 源创建单向绑定，而不是使用默认的 data context：

```csharp
var binding = CompiledBinding.Create<MyViewModel, string>(
    expression: vm => vm.Title,
    source: viewModel,
    mode: BindingMode.OneWay);
```

该表达式支持嵌套属性、索引器以及类型转换：

```csharp
// 嵌套属性
CompiledBinding.Create<MyViewModel, string>(vm => vm.Address.City);

// 带值转换器
CompiledBinding.Create<MyViewModel, bool>(
    expression: vm => vm.IsActive,
    converter: new BoolToOpacityConverter(),
    mode: BindingMode.OneWay);
```

你也可以在对象初始化器中使用它：

```csharp
var textBlock = new TextBlock
{
    [!TextBlock.TextProperty] = CompiledBinding.Create<MyViewModel, string>(
        expression: vm => vm.Name),
};
```

这种方式能让你在 C# 代码中获得与 XAML 编译绑定相同的性能和类型安全优势。对应的 XAML 写法请参阅 [Compiled Bindings](/docs/data-binding/compiled-bindings)。

## 订阅属性变化

你可以通过调用 [`GetObservable`](/api/avalonia/avaloniaobjectextensions#getobservable-method) 扩展方法，订阅 [`AvaloniaObject`](/api/avalonia/avaloniaobject) 的属性变化。该方法会返回一个 [`IObservable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.iobservable-1?view=net-10.0)，可用于监听属性值变化：

```csharp
var textBlock = new TextBlock();
var text = textBlock.GetObservable(TextBlock.TextProperty);
```

每个可订阅的属性都对应一个名为 `[PropertyName]Property` 的静态只读字段，你需要把它传给 `GetObservable`，以订阅该属性的变化。

[`IObservable<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.iobservable-1?view=net-10.0) 属于 Reactive Extensions（简称 rx）体系，本指南不展开介绍，但下面这个示例展示了如何使用返回的 observable，在属性变化时把值输出到控制台：

```csharp
var textBlock = new TextBlock();
var text = textBlock.GetObservable(TextBlock.TextProperty);
text.Subscribe(value => Console.WriteLine(value + " Changed"));
```

当你订阅返回的 observable 时，它会先立即返回当前属性值，之后每次属性变化时再推送新值。如果你不想收到当前值，可以使用 rx 的 `Skip` 操作符：

```csharp
var text = textBlock.GetObservable(TextBlock.TextProperty).Skip(1);
```

另一种方式是订阅 [`AvaloniaObject.PropertyChanged`](/api/avalonia/avaloniaobject#propertychanged-event) 事件。它会在该元素上的 _任意_ 属性发生变化时触发。

```csharp
textBlock.PropertyChanged += (s, e) =>
{
    if (e.Property == TextBlock.TextProperty)
    {
        Console.WriteLine(e.NewValue + " Changed");
    }
};
```

## 绑定到 observable

你可以使用 [`AvaloniaObject.Bind`](/api/avalonia/avaloniaobject#bind-method-1) 方法，把某个属性绑定到一个 observable：

```csharp
// 这里使用 Rx Subject，以便通过 OnNext 推送新值
var source = new Subject<string>();
var textBlock = new TextBlock();

// 将 TextBlock.Text 绑定到 source
var subscription = textBlock.Bind(TextBlock.TextProperty, source);

// 将 textBlock.Text 设为 "hello"
source.OnNext("hello");
// 将 textBlock.Text 设为 "world!"
source.OnNext("world!");

// 终止绑定
subscription.Dispose();
```

注意，`Bind` 方法会返回一个 `IDisposable`，你可以用它来终止绑定。如果你从不主动调用它，那么当 observable 通过 `OnCompleted` 或 `OnError` 结束时，绑定也会自动终止。

:::note
与标准的 Avalonia 绑定不同，observable 不使用弱引用，因此你需要自己负责管理其生命周期并防止内存泄漏。
:::

## 在对象初始化器中设置绑定

在对象初始化器中配置绑定通常很方便。你可以通过索引器语法来完成：

```csharp
var source = new Subject<string>();
var textBlock = new TextBlock
{
    Foreground = Brushes.Red,
    MaxWidth = 200,
    [!TextBlock.TextProperty] = CompiledBinding.Create<MyViewModel, string>(x => x.Title),
};
```

这个索引器也可以在对象初始化器之外使用：

```csharp
textBlock2[!TextBlock.TextProperty] = textBlock1[!TextBlock.TextProperty];
```

这种语法唯一的缺点是不会返回 `IDisposable`。如果你需要手动终止绑定，应当使用 `Bind` 方法。

## 在代码中使用反射绑定

要在代码中创建一个反射绑定：

```csharp
var binding = new ReflectionBinding("Name");
```

如果你希望获得编译期验证的类型安全绑定，应优先使用[编译绑定](#在代码中创建编译绑定)。

## 订阅任意对象上的属性

`GetObservable` 方法返回的是一个用于跟踪单个实例上某个属性变化的 observable。不过，如果你正在编写控件，可能更希望实现一个不依赖具体实例的 `OnPropertyChanged` 机制。

要做到这一点，你可以订阅 [`AvaloniaProperty.Changed`](/api/avalonia/avaloniaproperty)。这是一个 observable，只要 _任意实例_ 上的该属性发生变化，它就会触发。

> 在 WPF 中，这通常是通过在 `DependencyProperty` 注册时传入静态 `PropertyChangedCallback` 来实现的，但那样只有控件作者才能注册属性变化回调。

此外，还有一个 `AddClassHandler` 扩展方法，可以自动把这个事件路由到你的控件方法中。

例如，如果你想监听控件 `Foo` 属性的变化，可以这样写：

```csharp
static MyControl()
{
    FooProperty.Changed.AddClassHandler<MyControl>(FooChanged);
}

private static void FooChanged(MyControl sender, AvaloniaPropertyChangedEventArgs e)
{
    // 参数 e 描述了具体发生了什么变化。
}
```

## 检查当前活动绑定

你可以使用 `BindingOperations.GetBindingExpressionBase` 获取某个属性上当前活动的绑定表达式：

```csharp
var expression = BindingOperations.GetBindingExpressionBase(myTextBlock, TextBlock.TextProperty);
if (expression is not null)
{
    // TextBlock.TextProperty 上当前存在一个活动绑定
}
```

这对于诊断问题，或者在使用 `UpdateSourceTrigger.Explicit` 时手动调用 `UpdateSource()` 都很有帮助。

## 清除绑定

如果你保留了 `Bind` 返回的 `IDisposable`，可以通过调用 `Dispose` 来终止绑定：

```csharp
var subscription = textBlock.Bind(TextBlock.TextProperty, source);

// 稍后移除绑定
subscription.Dispose();
```

如果你手上没有 `IDisposable`（例如绑定是通过对象初始化器或 XAML 设置的），那么可以调用 `ClearValue` 来移除绑定，并让该属性回退到[优先级顺序](/docs/properties/value-precedence)中的下一个值：

```csharp
textBlock.ClearValue(TextBlock.TextProperty);
```

## 另请参阅

- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): XAML binding syntax reference.
- [Compiled Bindings](/docs/data-binding/compiled-bindings): Compile-time validated bindings.
- [Binding Debugging](/docs/data-binding/binding-debugging): Diagnosing binding issues.
- [Value Precedence](/docs/properties/value-precedence): How Avalonia resolves property values from multiple sources.
