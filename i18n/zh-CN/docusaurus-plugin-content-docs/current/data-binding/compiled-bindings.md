---
id: compiled-bindings
title: 编译绑定
description: 在 Avalonia XAML 中使用编译绑定，以获得编译期验证和更好的性能。
doc-type: how-to
---

在 XAML 中定义的绑定，默认会通过反射查找并访问 `ViewModel` 中请求的属性。在 Avalonia 中，你也可以使用编译绑定，它有几个明显优势：

* 如果你使用了编译绑定，而要绑定的属性不存在，那么会直接得到编译错误。这会显著提升调试体验。
* 众所周知，反射性能相对较慢，因此使用编译绑定可以提升应用程序性能。

## 启用和禁用编译绑定

:::info

根据你创建 Avalonia 项目时所使用的模板不同，编译绑定可能默认启用，也可能默认未启用。你可以在项目文件中检查这一点。

::: 

### 全局启用与禁用

如果你希望整个应用程序默认全局使用编译绑定，请在项目文件中添加：

```xml
<AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
```

你仍然需要为要绑定的对象提供 `x:DataType`，但不再需要为每个 `UserControl` 或 `Window` 单独设置 `x:CompileBindings="[True|False]"`。

### 按 UserControl 或 Window 启用与禁用

要启用编译绑定，首先需要定义你想绑定对象的 `DataType`。在 [`DataTemplates`](/docs/data-templates/introduction-to-data-templates) 中有一个 `DataType` 属性；对于其他元素，则可以通过 `x:DataType` 来设置。通常你会在根节点上设置 `x:DataType`，例如在 `Window` 或 `UserControl` 上。你也可以直接在 `Binding` 中指定 `DataType`。

然后你就可以通过设置 `x:CompileBindings="[True|False]"` 来启用或禁用编译绑定。所有子节点都会继承这个属性，因此你可以在根节点启用它，并在需要时对某个子节点单独禁用。

```xml
<!-- 设置 DataType 并启用编译绑定 -->
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="using:MyApp.ViewModels"
             x:DataType="vm:MyViewModel"
             x:CompileBindings="True">
    <StackPanel>
        <TextBlock Text="Last name:" />
        <TextBox Text="{Binding LastName}" />
        <TextBlock Text="Given name:" />
        <TextBox Text="{Binding GivenName}" />
        <TextBlock Text="E-Mail:" />
        <!-- 在 Binding 标记中设置 DataType -->
        <TextBox Text="{Binding MailAddress, DataType={x:Type vm:MyViewModel}}" />

        <Button Content="Send an E-Mail"
                Command="{Binding SendEmailCommand}" />
    </StackPanel>
</UserControl>
```

## `DataContext` 类型推断

当启用了编译绑定，并且在根元素上设置了 `x:DataType` 后，Avalonia XAML 编译器就能够推断绑定目标类型，即便你是通过命名元素（`#MyElement.DataContext`）或父级查找（`$parent[ControlType].DataContext`）来引用它也没问题。

在大多数情况下，你都不需要显式类型转换。

例如：

```xml
<Window x:Name="MyWindow"
        xmlns:vm="using:MyApp.ViewModels"
        x:DataType="vm:TestDataContext">
    <TextBlock Text="{Binding #MyWindow.DataContext.StringProperty}" />
    <TextBlock Text="{Binding $parent[Window].DataContext.StringProperty}" />
</Window>
```

:::note
`DataContext` 类型推断是在 11.3.0 中引入的。在更早版本的 Avalonia 中，如果绑定表达式的目标类型无法自动判断，就需要显式地进行类型转换。
:::

:::note
如果你使用 [Rider](https://www.jetbrains.com/rider/) 作为 IDE，语法高亮可能会错误地提示异常，不过编译器通常仍能正常工作。
:::

### 显式类型转换

如果你使用的是较早版本的 Avalonia，或者编译器未能正确推断类型，那么仍然可以在绑定表达式中显式进行类型转换，以确保使用正确类型。

通常并不推荐显式类型转换，除非确实有必要。

```xml
<Window x:Name="MyWindow"
        xmlns:vm="using:MyApp.ViewModels"
        x:DataType="vm:TestDataContext">
    <TextBlock Text="{Binding #MyWindow.((vm:TestDataContext)DataContext).StringProperty}" />
    <TextBlock Text="{Binding $parent[Window].((vm:TestDataContext)DataContext).StringProperty}" />
</Window>
```

## `CompiledBinding` 标记

如果你不想让所有子节点都启用编译绑定，也可以单独使用 `CompiledBinding` 标记。你仍然需要定义 `DataType`，但可以省略 `x:CompileBindings="True"`。

```xml
<!-- 设置 DataType -->
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="using:MyApp.ViewModels"
             x:DataType="vm:MyViewModel">
    <StackPanel>
        <TextBlock Text="Last name:" />
        <!-- 在绑定中使用 CompiledBinding 标记 -->
        <TextBox Text="{CompiledBinding LastName}" />
        <TextBlock Text="Given name:" />
        <TextBox Text="{CompiledBinding GivenName}" />
        <TextBlock Text="E-Mail:" />
        <TextBox Text="{CompiledBinding MailAddress}" />

        <!-- 这个命令会使用默认的 ReflectionBinding -->
        <Button Content="Send an E-Mail"
                Command="{Binding SendEmailCommand}" />
    </StackPanel>
</UserControl>
```

## `ReflectionBinding` 标记

如果你已经在根节点上启用了编译绑定（通过 `x:CompileBindings="True"`），但在某个位置不想使用编译绑定，就可以改用 `ReflectionBinding` 标记。

```xml
<!-- 设置 DataType -->
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="using:MyApp.ViewModels"
             x:DataType="vm:MyViewModel"
             x:CompileBindings="True">
    <StackPanel>
        <TextBlock Text="Last name:" />
        <TextBox Text="{Binding LastName}" />
        <TextBlock Text="Given name:" />
        <TextBox Text="{Binding GivenName}" />
        <TextBlock Text="E-Mail:" />
        <TextBox Text="{Binding MailAddress}" />

        <!-- 这里改为使用 ReflectionBinding -->
        <Button Content="Send an E-Mail"
                Command="{ReflectionBinding SendEmailCommand}" />
    </StackPanel>
</UserControl>
```

## 在代码中使用编译绑定

你也可以在 C# 代码中使用 `CompiledBinding.Create` 工厂方法创建编译绑定。这样可以像 XAML 编译绑定一样获得编译期安全性和性能优势，只不过这里使用的是 LINQ 表达式，而不是字符串形式的属性路径。示例请参阅 [Compiled bindings from code](/docs/data-binding/binding-from-code#creating-compiled-bindings-from-code)。

## 另请参阅

- [Compiled bindings from code](/docs/data-binding/binding-from-code#creating-compiled-bindings-from-code)
- [Data binding syntax](/docs/data-binding/data-binding-syntax)
