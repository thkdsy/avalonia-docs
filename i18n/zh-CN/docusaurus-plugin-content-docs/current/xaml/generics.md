---
id: generics
title: XAML 中的泛型类型
description: 通过 x:TypeArguments 指令在 Avalonia XAML 中使用泛型 .NET 类型，包括集合与自定义控件。
doc-type: concept
---

Avalonia 支持通过 `x:TypeArguments` 指令在 XAML 中使用泛型 .NET 类型。这使你能够直接在标记中实例化泛型类并使用泛型集合。

## `x:TypeArguments`

`x:TypeArguments` 指令用于为泛型类型指定类型参数。它只能用于 XAML 文件的根元素，或者同时具有 `x:Class` 的元素，或位于资源字典内部的元素。

### 基本语法

```xml
<local:MyGenericControl x:TypeArguments="x:String" />
```

### 多个类型参数

多个类型参数之间使用逗号分隔：

```xml
<local:Pair x:TypeArguments="x:String, x:Int32" />
```

## 在资源中使用泛型集合

你可以把泛型集合作为资源来定义：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:sys="using:System"
        xmlns:scg="using:System.Collections.Generic">

    <Window.Resources>
        <scg:List x:Key="Colors" x:TypeArguments="sys:String">
            <sys:String>Red</sys:String>
            <sys:String>Blue</sys:String>
            <sys:String>Green</sys:String>
        </scg:List>
    </Window.Resources>

    <ListBox ItemsSource="{StaticResource Colors}" />
</Window>
```

## 泛型自定义控件

创建泛型自定义控件时，需要先在 C# 中定义类型参数：

```csharp
public class TypedList<T> : ItemsControl
{
    public static readonly StyledProperty<T?> SelectedValueProperty =
        AvaloniaProperty.Register<TypedList<T>, T?>(nameof(SelectedValue));

    public T? SelectedValue
    {
        get => GetValue(SelectedValueProperty);
        set => SetValue(SelectedValueProperty, value);
    }
}
```

然后在 XAML 中配合 `x:TypeArguments` 使用它：

```xml
<local:TypedList x:TypeArguments="vm:Person" ItemsSource="{Binding People}" />
```

## XAML 中常见的泛型类型

| 类型 | XAML 前缀 | 示例 |
|---|---|---|
| `System.String` | `x:String` | `x:TypeArguments="x:String"` |
| `System.Int32` | `x:Int32` | `x:TypeArguments="x:Int32"` |
| `System.Double` | `x:Double` | `x:TypeArguments="x:Double"` |
| `System.Boolean` | `x:Boolean` | `x:TypeArguments="x:Boolean"` |
| 自定义类型 | `local:` 或 `vm:` | `x:TypeArguments="vm:MyModel"` |

## 限制

- 根元素上的 `x:TypeArguments` 需要同时指定 `x:Class`。
- XAML 不支持嵌套泛型类型（例如 `List<List<string>>`）。可以在代码中定义后，再通过绑定或 `x:Static` 引用。
- 并非所有 XAML 上下文都支持 `x:TypeArguments`。它适用于对象元素和资源定义。

## 不受支持场景的替代方案

当 XAML 泛型不适用时，可以定义具体的子类：

```csharp
// 定义一个可在 XAML 中使用的非泛型子类
public class StringList : List<string> { }
public class PersonCollection : ObservableCollection<Person> { }
```

```xml
<!-- 直接使用具体类型 -->
<local:StringList x:Key="Names">
    <sys:String>Alice</sys:String>
    <sys:String>Bob</sys:String>
</local:StringList>
```

这种模式在 .NET XAML 框架中很常见，也可以避开 `x:TypeArguments` 的各种限制。

## 另请参阅

- [XAML 命名空间](/docs/xaml/namespaces)：了解如何在 XAML 中引用 CLR 命名空间。
- [x: 指令](/docs/xaml/directives)：`x:TypeArguments` 及其他指令的完整参考。
- [类型转换器](/docs/xaml/type-converters)：将字符串值转换为 .NET 类型。
