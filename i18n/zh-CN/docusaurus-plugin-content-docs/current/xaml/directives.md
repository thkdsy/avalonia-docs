---
id: directives
title: "x: 指令"
---

XAML 指令是 `x:` 命名空间中的特殊属性，用于控制 XAML 引擎如何处理元素。它们属于 XAML 语言规范的一部分，并不特定于某个具体控件。

要使用这些指令，你需要先声明 XAML 语言命名空间：

```xml
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
```

## `x:Class`

将 XAML 文件连接到其代码后置类。这个指令必须放在 XAML 文件的根元素上。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="MyApp.MainWindow">
</Window>
```

指定的类必须是一个 `partial` 类，并且继承自根元素的类型：

```csharp
namespace MyApp;

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }
}
```

## `x:Name`

为元素分配名称，并在代码后置类中生成对应字段，这样你就可以在 C# 中引用该控件。

```xml
<TextBox x:Name="SearchBox" PlaceholderText="搜索..." />
<Button Content="搜索" Click="OnSearchClick" />
```

```csharp
private void OnSearchClick(object? sender, RoutedEventArgs e)
{
    var query = SearchBox.Text;
    // 使用已命名的控件
}
```

:::info
对于大多数 Avalonia 控件来说，`x:Name` 与 `Name` 可以互换使用。`Name` 属性存在于 `StyledElement` 上，并设置相同的底层值。当元素类型本身没有 `Name` 属性时，请使用 `x:Name`。
:::

## `x:Key`

为资源分配一个字典键，可用于 `ResourceDictionary`、`Styles` 或 `Application.Resources`：

```xml
<Application.Resources>
    <SolidColorBrush x:Key="PrimaryBrush" Color="#1976D2" />
    <x:Double x:Key="DefaultSpacing">8</x:Double>
</Application.Resources>
```

资源通常通过 `{StaticResource}` 或 `{DynamicResource}` 获取：

```xml
<Border Background="{StaticResource PrimaryBrush}" Padding="{StaticResource DefaultSpacing}" />
```

## `x:DataType`

指定某个作用域内数据绑定的预期数据类型。这是 [编译绑定](/docs/data-binding/compiled-bindings) 所必需的，同时还能为绑定路径提供 IntelliSense 支持。

```xml
<Window x:DataType="vm:MainWindowViewModel">
    <TextBlock Text="{Binding UserName}" />
</Window>
```

在 `DataTemplate` 上的用法：

```xml
<DataTemplate x:DataType="vm:TodoItemViewModel">
    <StackPanel>
        <CheckBox IsChecked="{Binding IsComplete}" />
        <TextBlock Text="{Binding Title}" />
    </StackPanel>
</DataTemplate>
```

## `x:CompileBindings`

为当前作用域中的所有绑定启用或禁用编译绑定。编译绑定会在编译时进行验证，并且通常具有更好的性能。

```xml
<UserControl x:CompileBindings="True"
             x:DataType="vm:MyViewModel">
    <!-- 此处所有绑定都会编译 -->
    <TextBlock Text="{Binding Name}" />
</UserControl>
```

你也可以在项目文件中全局设置它：

```xml
<AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
```

如果要让某个特定绑定退出编译绑定机制，可以使用 `ReflectionBinding`：

```xml
<TextBlock Text="{ReflectionBinding DynamicProperty}" />
```

## `x:Static`

引用静态字段、属性、常量或枚举值：

```xml
<TextBlock Text="{x:Static sys:Environment.MachineName}" />

<Rectangle Fill="{x:Static Brushes.Red}" />

<Border Width="{x:Static local:Constants.DefaultWidth}" />
```

用于枚举值时：

```xml
<ComboBox SelectedItem="{x:Static local:Priority.High}" />
```

## `x:Type`

引用一个 `System.Type` 对象：

```xml
<Style Selector="Button">
    <Setter Property="Tag" Value="{x:Type Button}" />
</Style>
```

## `x:Null`

将某个属性设置为 `null`：

```xml
<Button Background="{x:Null}" Content="No background" />
```

## `x:True` 与 `x:False`

布尔值的简写形式。这是 Avalonia 特有的扩展：

```xml
<CheckBox IsChecked="{x:True}" />
<TextBox IsReadOnly="{x:False}" />
```

它们等价于：

```xml
<CheckBox IsChecked="True" />
<TextBox IsReadOnly="False" />
```

## `x:Shared`

控制资源是只实例化一次并重复使用，还是每次被引用时都重新创建。默认情况下资源是共享的（每次都会返回同一个实例）。将 `x:Shared="False"` 设置为关闭共享后，每次引用都会创建新实例：

```xml
<Application.Resources>
    <ColumnDefinitions x:Key="TwoColumnLayout" x:Shared="False">
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="Auto" />
    </ColumnDefinitions>
</Application.Resources>
```

如果不设置 `x:Shared="False"`，把同一个 `ColumnDefinitions` 资源分配给多个 `Grid` 控件会失败，因为同一个实例不能同时拥有多个父级。

:::info
`x:Shared` 只对 `ResourceDictionary` 中的资源生效。在资源定义之外使用它不会有任何效果。
:::

## 基元类型元素

XAML 语言命名空间为常见的 .NET 基元类型提供了对应元素：

```xml
<x:String>你好，世界</x:String>
<x:Double>3.14</x:Double>
<x:Int32>42</x:Int32>
<x:Boolean>True</x:Boolean>
```

这些类型在定义资源时非常有用：

```xml
<Application.Resources>
    <x:Double x:Key="HeaderFontSize">24</x:Double>
    <x:String x:Key="AppTitle">我的应用</x:String>
</Application.Resources>
```

## 另请参阅

- [XAML 参考](/docs/xaml)：XAML 语法总览。
- [命名空间](/docs/xaml/namespaces)：XAML 命名空间的工作方式。
- [标记扩展](/docs/xaml/markup-extensions)：`{Binding}`、`{StaticResource}` 及其他扩展。
- [编译绑定](/docs/data-binding/compiled-bindings)：编译绑定的工作原理。
