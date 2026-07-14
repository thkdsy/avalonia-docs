---
id: markup-extensions
title: 标记扩展
description: 创建自定义 XAML 标记扩展，在运行时为属性提供值。
doc-type: how-to
---

<p>{frontMatter.description}</p>

## 关于标记扩展

经典的标记扩展通常是指满足以下条件的任意类：

- 实现了 `object? ProvideValue(IServiceProvider?)`
- 可以选择继承 [`MarkupExtension`](/api/avalonia/markup/xaml/markupextension)（在 Avalonia 中不是必需的）
- 通过 `{ns:Extension ...}` 这种语法在 XAML 中使用

在 Avalonia 中，`ProvideValue` 允许返回 **任意** 类型。这意味着返回结果可以是强类型的，因为该返回值会直接赋给目标属性。

Avalonia 提供了以下标记扩展：

| MarkupExtension | 赋值到的属性含义 |
|--------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| [StaticResource](/docs/app-development/resource-dictionary#static-resource) | 引用一个已存在的键值资源，并且在资源变化时不会更新 |
| [DynamicResource](/docs/app-development/resource-dictionary#using-resources) | 延迟加载某个键值资源，并在资源变化时自动更新 |
| Binding | 根据默认绑定偏好决定：Compiled 或 Reflection |
| [CompiledBinding](/docs/data-binding/compiled-bindings#compiledbinding-markup) | 基于编译绑定 |
| [ReflectionBinding](/docs/data-binding/compiled-bindings#reflectionbinding-markup) | 基于反射绑定 |
| [TemplateBinding](/docs/custom-controls/templated-controls) | 基于一种简化绑定，仅在 `ControlTemplate` 中使用 |
| [OnPlatform](/docs/platform-specific-guides/xaml#onplatform-markup-extension) | 仅在指定平台上条件生效 |
| [OnFormFactor](/docs/platform-specific-guides/xaml#onformfactor-markup-extension) | 仅在指定设备形态下条件生效 |

## 编译器内建语法

严格来说，这些内容属于 XAML 编译器的一部分，而不算真正的 `MarkupExtension`，但它们的 XAML 语法看起来是一样的。

| 内建项 | 表示内容 |
|-----------|-----------------------|
| x:True | `true` 字面量 |
| x:False | `false` 字面量 |
| x:Null | `null` 字面量 |
| x:Static | 静态成员值 |
| x:Type | `System.Type` 字面量 |

`x:True` 和 `x:False` 在某些场景下很有用，例如目标绑定属性是 `object`，但你需要传入一个布尔值。在这类缺乏类型信息的场景中，直接写 `"True"` 仍然会被视为 `string`。

```xml
<Button Command="{Binding SetStateCommand}" CommandParameter="{x:True}" />
```

## 创建标记扩展

你可以继承 `MarkupExtension`，或者实现以下任意一种方法签名（Avalonia 会通过 duck-typing 识别它们）：

```csharp
T ProvideValue();
T ProvideValue(IServiceProvider provider);
object ProvideValue();
object ProvideValue(IServiceProvider provider);
```

下面是一个用于本地化的基础标记扩展示例：

```csharp
public class LocExtension
{
    public string Key { get; set; } = "";

    public string ProvideValue(IServiceProvider serviceProvider)
    {
        // 简化版本地化查找
        return LocalizationService.GetString(Key) ?? Key;
    }
}
```

```xml
<TextBlock Text="{local:Loc Key=WelcomeMessage}" />
```

如果你使用的是强类型返回值而不是 `object`，那么当 XAML 中传入的构造函数参数、属性或 `ProvideValue` 返回值类型不匹配时，就会直接在编译期报错。若返回的是 `object`，那么实际返回类型必须与目标属性类型一致，否则运行时会抛出 `InvalidCastException`。

### 使用 `IServiceProvider`

传递给 `ProvideValue` 的 `IServiceProvider` 会暴露出 XAML 上下文服务，使标记扩展能够知道自己被用在什么位置。

常见的标准服务包括：

- **`IProvideValueTarget`**：提供对目标对象和目标属性的访问。
- **`IRootObjectProvider`**：提供 XAML 文档的根对象。

Avalonia 还提供了一些额外的、XAML-IL 特有的服务：

- **`IAvaloniaXamlIlParentStackProvider`**：在 XAML 解析过程中暴露父对象栈。
- **`IAvaloniaXamlIlXmlNamespaceInfoProvider`**：提供命名空间元数据。

这些服务并非总是必需，但对于更高级、依赖上下文的扩展来说非常重要。

### 接收字面量参数

如果需要强制传入参数，可以通过构造函数按顺序接收这些参数。

对于可选参数或无序参数，则可以使用属性。你也可以混合使用多个构造函数，包括无参构造函数。

```csharp
public class MultiplyLiteral
{
    private readonly double _first;
    private readonly double _second;
    
    public double? Third { get; set; }

    public MultiplyLiteral(double first, double second)
    {
        _first = first;
        _second = second;
    }

    public double ProvideValue(IServiceProvider provider)
    {
        return First * Second * Third ?? 1;
    }
}
```
```xml
<TextBlock Text="This has FontSize=40" FontSize="{namespace:MultiplyLiteral 10, 8, Third=0.5}" />
```

### 从绑定中接收参数

一种常见场景是：从绑定中接收数据，进行变换后再更新目标属性。当所有参数都来自绑定时，可以通过创建一个带有 `IMultiValueConverter` 的 `MultiBinding` 来实现。

在下面的示例中，`MultiplyBinding` 需要两个绑定参数。如果你既要支持字面量参数，又要支持绑定参数，那么创建一个 `IMultiValueConverter` 会更灵活，因为你可以通过构造函数参数或 `init` 属性传递字面量。`BindingBase` 允许同时使用 `CompiledBinding` 和 `ReflectionBinding`，但本身不支持直接传入字面量。

```csharp
public class MultiplyBinding
{
    private readonly BindingBase _first;
    private readonly BindingBase _second;

    public MultiplyBinding(BindingBase first, BindingBase second)
    {
        _first = first;
        _second = second;
    }

    public object ProvideValue()
    {
        var mb = new MultiBinding()
        {
            Bindings = new[] { _first, _second },
            Converter = new FuncMultiValueConverter<double, double>(doubles => doubles.Aggregate(1d, (x, y) => x * y))
        };

        return mb;
    }
}
```

```xml
<TextBlock FontSize="{local:MultiplyBinding {Binding Multiplier}, {Binding Multiplicand}}" 
           Text="MarkupExtension with Bindings!" />
```

:::info
另一种做法是直接返回 `IObservable<T>.ToBinding()`。
:::

### 返回值类型

Avalonia 的标记扩展模型非常灵活：`ProvideValue` 可以返回任何东西。

这包括：

- 静态 .NET 对象
- 强类型 .NET 对象，这样在赋值到属性时可以获得编译期验证
- **Binding** 实例
- 用于动态响应式值的 **Observables（`IObservable<T>`）**

返回 Binding 或 Observable 的标记扩展都是受支持的，并且能够与 Avalonia 的属性系统和数据绑定系统集成。

如果你希望一个标记扩展兼容多个目标属性类型，可以让 `ProvideValue` 返回 `object`，这样就能在内部针对不同类型做单独处理。


```csharp
public object ProvideValue(IServiceProvider provider)
{
    var target = (IProvideValueTarget)provider.GetService(typeof(IProvideValueTarget))!;
    var targetProperty = target.TargetProperty as AvaloniaProperty;
    var targetType = targetProperty?.PropertyType;

    double result = First * Second * (Third ?? 1);

    if (targetType == typeof(double))
        return result;
    else if (targetType == typeof(float))
        return (float)result;
    else if (targetType == typeof(int))
        return (int)result;
    else
        throw new NotSupportedException();
}
```

构造函数同样也可以用 `object` 的方式接收参数类型，但相应地，原本可以在编译期发现的问题，也会变成运行时异常。

### MarkupExtension 属性特性

* `[ConstructorArgument]` - 表示关联属性可以通过构造函数参数初始化；如果实际使用了构造函数，那么在 XAML 序列化时应忽略该属性。
* `[MarkupExtensionOption]`、`[MarkupExtensionDefaultOption]` - 与 `ShouldProvideOption` 配合使用；可以参考 `OnPlatform` 和 `OnFormFactor` 的源码示例。

## 选项型标记扩展

`OptionsMarkupExtension` 是一种特殊的标记扩展，专门用于类似 switch 的表达式场景。它的目标是通过移除永远不会用到的分支来实现优化，从而让编译器可以进一步裁剪代码。

### `OnPlatform` 标记扩展

内置的 `OnPlatform` 标记扩展就是选项型标记扩展的一个例子。它会针对不同运行平台（Windows、macOS、Linux 等）定义不同的值，从而优化分支，只保留对当前编译目标平台有意义的部分。

With `OnPlatform`, you can, for instance, use the `Markdown` control on Linux and the `WebView` control on other platforms. The unused control would be excluded, thus reducing the binary size.

### Creating custom options markup extensions

Here is an example of a custom implementation with `RuntimeInformation.ProcessArchitecture`. As shown in this example, we recommend using compiler flags or .NET runtime APIs that are effectively constant.

```csharp
public class ArchitectureExtension : IAddChild<On<object>>
{
    [MarkupExtensionOption(nameof(X86))] public object? X86 { get; set; }
    [MarkupExtensionOption(nameof(X64))] public object? X64 { get; set; }
    [MarkupExtensionOption(nameof(Arm))] public object? Arm { get; set; }
    [MarkupExtensionOption(nameof(Arm64))] public object? Arm64 { get; set; }
    [MarkupExtensionOption(nameof(Wasm))] public object? Wasm { get; set; }

    [Content]
    [MarkupExtensionDefaultOption]
    public object? Default { get; set; }

    public static bool ShouldProvideOption(string option)
    {
        var currentArch = RuntimeInformation.ProcessArchitecture;
        return option switch
        {
            nameof(X86) => currentArch == Architecture.X86,
            nameof(X64) => currentArch == Architecture.X64,
            nameof(Arm) => currentArch == Architecture.Arm,
            nameof(Arm64) => currentArch == Architecture.Arm64,
            nameof(Wasm) => currentArch == Architecture.Wasm,
            _ => false,
        };
    }

    // Needed for the compiler.
    public void AddChild(On<object> child) {}
    public object? ProvideValue() => null;
}
```

This class defines several options that are selected through the `ShouldProvideOption` static method. You can then set the options in XAML, like so:

```xml
<Border Background="{local:Architecture Default=White, X64=Green, Arm64=Red, Wasm=Blue}" />
```

The example above, in a non-optimized .NET build, is equivalent to the following code.

```csharp
border.Background = ArchitectureExtension.ShouldProvideOption("X64") ? Brushes.Green
     : ArchitectureExtension.ShouldProvideOption("Arm64") ? Brushes.Red
     : ArchitectureExtension.ShouldProvideOption("Wasm") ? Brushes.Blue
     : Brushes.White;
```

Once optimized and trimmed for specific platform architecture, it is reduced to the following instead.

```csharp
border.Background = Brushes.Red; // assuming app was compiled with dotnet publish -r win-arm64;
```

## See also

- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding MarkupExtension reference.
- [Compiled Bindings](/docs/data-binding/compiled-bindings): CompiledBinding and ReflectionBinding markup.
- [Platform-specific XAML](/docs/platform-specific-guides/xaml): OnPlatform and OnFormFactor markup extensions.
