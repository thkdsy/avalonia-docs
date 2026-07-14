---
id: index
title: XAML 参考
---

本节提供 Avalonia 中可用的 XAML 语言特性的参考说明。虽然 [Avalonia XAML 基础](/docs/fundamentals/avalonia-xaml) 页面介绍了基本概念，但这里会更深入地说明语法、指令、标记扩展以及 XAML 编译流程。

## 什么是 XAML？

XAML（eXtensible Application Markup Language，可扩展应用程序标记语言）是一种基于 XML 的语言，用于声明对象图。在 Avalonia 中，XAML 用于以声明方式定义用户界面。每个 XML 元素都会映射到一个 .NET 对象，而 XML 属性则用于设置这些对象的属性。

Avalonia 使用 `.axaml` 文件扩展名（Avalonia XAML）来区分它的 XAML 文件与 WPF 或其他 XAML 方言。这可以避免在 Visual Studio 和其他工具中发生冲突。

## XAML 语法

### 对象元素

一个 XML 元素会创建对应类型的实例：

```xml
<Button />
<TextBlock />
<StackPanel />
```

### 属性特性

使用 XML 属性来设置属性值：

```xml
<Button Content="Click me" Width="200" Background="Blue" />
```

XAML 引擎会使用 [类型转换器](/docs/xaml/type-converters) 将字符串形式的属性值转换为适当的 .NET 类型（例如，`"Blue"` 会变成一个 `SolidColorBrush`）。

### 属性元素语法

对于无法用字符串表达的复杂值，可以使用属性元素语法：

```xml
<Button>
    <Button.Background>
        <LinearGradientBrush StartPoint="0%,0%" EndPoint="100%,100%">
            <GradientStop Color="Red" Offset="0" />
            <GradientStop Color="Blue" Offset="1" />
        </LinearGradientBrush>
    </Button.Background>
    <Button.Content>
        <StackPanel Orientation="Horizontal">
            <Image Source="/Assets/icon.png" Width="16" Height="16" />
            <TextBlock Text="Click me" Margin="4,0,0,0" />
        </StackPanel>
    </Button.Content>
</Button>
```

### 内容属性

许多控件都会指定一个默认内容属性。直接放在控件标签内部的子元素会被赋值给该属性：

```xml
<!-- 这两种写法等价 -->
<Button>Click me</Button>
<Button Content="Click me" />
```

```xml
<!-- StackPanel 的内容属性是 Children -->
<StackPanel>
    <TextBlock Text="First" />
    <TextBlock Text="Second" />
</StackPanel>
```

### 集合语法

集合类型的属性可以通过多个子元素来填充：

```xml
<Grid.ColumnDefinitions>
    <ColumnDefinition Width="Auto" />
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="200" />
</Grid.ColumnDefinitions>
```

某些集合属性支持简洁的字符串写法：

```xml
<Grid ColumnDefinitions="Auto,*,200" RowDefinitions="Auto,*" />
```

### 附加属性语法

使用 `OwnerType.PropertyName` 语法来设置附加属性：

```xml
<Grid>
    <Button Grid.Row="0" Grid.Column="1" Content="Cell (0,1)" />
</Grid>
```

## 主题

- [命名空间](/docs/xaml/namespaces)：了解 XAML 命名空间的工作方式，以及如何引用你自己的类型。
- [x: 指令](/docs/xaml/directives)：`x:Name`、`x:Key`、`x:Class`、`x:DataType` 等指令的参考说明。
- [标记扩展](/docs/xaml/markup-extensions)：`{Binding}`、`{StaticResource}`、`{DynamicResource}`、`{TemplateBinding}` 等扩展的参考说明。
- [类型转换器](/docs/xaml/type-converters)：了解 XAML 中的字符串值如何转换为 .NET 类型。

## 另请参阅

- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)：XAML 基础与文件结构。
- [数据绑定](/docs/data-binding/introduction-to-data-binding)：数据绑定参考。
- [样式](/docs/styling/styles)：Avalonia 中类似 CSS 的样式系统。
