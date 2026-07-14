---
id: compilation
title: XAML 编译
---

Avalonia 使用 XamlX 编译器在构建时处理 `.axaml` 文件。与默认在运行时解释 XAML 的 WPF 不同，Avalonia 会在构建期间把 XAML 编译成 IL 代码，从而带来更快的启动速度、更小的二进制体积，以及编译期错误检测能力。

## XAML 编译如何工作

当你构建 Avalonia 项目时，XamlX 编译器会：

1. 解析每个 `.axaml` 文件。
2. 解析所有类型引用、命名空间和属性名称。
3. 验证属性赋值和类型转换。
4. 生成可直接构建视觉树的 IL 代码，而无需在运行时解析 XML。

这意味着许多在 WPF 中只会在运行时出现的错误，会在 Avalonia 的编译阶段就被捕获。

## 编译绑定

默认情况下，数据绑定会在运行时通过反射解析属性路径。编译绑定则会在构建时完成路径解析，并提供以下优势：

- **构建时验证**：属性名拼写错误会直接导致编译器报错。
- **更好的性能**：运行时没有反射开销。
- **AOT 兼容性**：Native AOT 部署需要它。

### 为单个控件启用编译绑定

使用 `x:DataType` 声明数据类型，并通过 `x:CompileBindings` 启用编译：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="using:MyApp.ViewModels"
             x:DataType="vm:MainViewModel"
             x:CompileBindings="True">
    <!-- 此绑定会在编译时验证 -->
    <TextBlock Text="{Binding UserName}" />
</UserControl>
```

如果 `MainViewModel` 上不存在 `UserName`，构建就会失败并报错。

### 在整个项目中启用编译绑定

将以下属性加入 `.csproj` 文件，即可让编译绑定成为默认行为：

```xml
<PropertyGroup>
    <AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
</PropertyGroup>
```

启用此设置后，所有绑定都需要声明 `x:DataType`。对于某些必须在运行时解析的特定绑定，可以使用 [`ReflectionBinding`](/api/avalonia/data/reflectionbinding) 退出该机制：

```xml
<TextBlock Text="{ReflectionBinding DynamicProperty}" />
```

### 在嵌套元素上使用 x:DataType

你可以在模板或嵌套作用域中切换数据类型：

```xml
<ListBox ItemsSource="{Binding Orders}">
    <ListBox.ItemTemplate>
        <DataTemplate x:DataType="vm:OrderViewModel">
            <TextBlock Text="{Binding OrderNumber}" />
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

## 构建期错误示例

启用编译式 XAML 绑定后，以下常见错误都会变成构建错误：

| 错误写法 | 报错 |
|---|---|
| `{Binding UserNam}`（拼写错误） | Cannot resolve property 'UserNam' on type 'MainViewModel' |
| 缺少 `x:DataType` | Cannot use compiled binding without a DataType |
| 属性类型错误 | Cannot assign 'string' to property of type 'int' |

## Native AOT 注意事项

当目标是 Native AOT 时，必须使用编译绑定，因为基于反射的绑定在缺少完整运行时的情况下可能无法工作。请确保：

1. `.csproj` 中的 `AvaloniaUseCompiledBindingsByDefault` 为 `true`。
2. 所有绑定都具有对应的 `x:DataType` 声明。
3. 不再使用 `ReflectionBinding`（或者已通过合适的裁剪器注解进行保护）。

关于 AOT 部署的更多细节，请参阅 [Native AOT](/docs/deployment/native-aot)。

## 过时与实验性诊断

XAML 编译器能够识别类型和成员上的 `[Obsolete]` 与 `[Experimental]` 特性。当你在 XAML 中使用已过时或实验性的类型、属性或事件时，编译器会发出相应的警告；如果 `[Obsolete]` 使用了 `error: true`，则会直接报错，并附带对应的诊断代码和消息。

例如，如果某个控件库将一个类型标记为实验性：

```csharp
[Experimental("MYLIB0001")]
public class PreviewPanel : Control { }
```

那么在 XAML 中使用它时会产生构建警告：

```text
warning MYLIB0001: 'PreviewPanel' is for evaluation purposes only and is subject to change or removal in future updates.
  --> Views/MainView.axaml(8,6)
```

类似地，如果在 XAML 中使用了标记为 `[Obsolete("Use NewProperty instead")]` 的成员，就会在构建时发出 `AVLN2001` 警告，而不是静默通过编译。

如果需要，你可以在项目文件中屏蔽这些诊断：

```xml
<PropertyGroup>
    <NoWarn>$(NoWarn);MYLIB0001</NoWarn>
</PropertyGroup>
```

## XAML 编译故障排查

### XAML 文件中的构建错误

XAML 编译错误会显示在 IDE 的错误列表和构建输出中，并附带文件名与行号：

```text
error AVLN2000: Unable to resolve property 'Naem' on type 'MyApp.ViewModels.PersonViewModel'
  --> Views/PersonView.axaml(12,34)
```

### 为动态场景关闭编译错误

如果你在某些特定场景下需要运行时解析绑定（例如绑定到 `dynamic` 或 `ExpandoObject`），请使用 `ReflectionBinding`：

```xml
<TextBlock Text="{ReflectionBinding SomeDynamicProperty}" />
```

### 设计时数据类型

为了让 XAML 预览器和 IntelliSense 正常工作，请确保设置了 `x:DataType`。这也会在支持该功能的 IDE 中为绑定路径启用自动补全。

```xml
<UserControl x:DataType="vm:MainViewModel">
    <!-- IDE 会为绑定路径提供 IntelliSense -->
    <TextBox Text="{Binding SearchText}" />
</UserControl>
```

## 另请参阅

- [编译绑定](/docs/data-binding/compiled-bindings)：更详细的编译绑定参考。
- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)：XAML 基础。
- [x: 指令](/docs/xaml/directives)：`x:CompileBindings`、`x:DataType` 等指令的完整参考。
- [Native AOT](/docs/deployment/native-aot)：AOT 部署指南。
