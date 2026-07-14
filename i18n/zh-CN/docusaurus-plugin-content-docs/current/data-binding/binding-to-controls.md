---
id: binding-to-controls
title: 如何绑定到控件
description: 使用 ElementName 或源绑定，将一个控件的属性绑定到另一个控件的属性。
doc-type: how-to
---


在 _Avalonia UI_ 中，除了绑定到 data context 之外，你也可以直接把一个控件绑定到另一个控件。

:::info
请注意，这种技术完全不使用 data context。也就是说，你绑定的是另一个控件本身。
:::

## 绑定到具名控件

如果你想绑定到另一个已命名控件的属性，可以在控件名前加上 `#` 字符。

```xml
<TextBox Name="other">

<!-- 绑定到名为 "other" 的控件的 Text 属性 -->
<TextBlock Text="{Binding #other.Text}"/>
```

这与 WPF 和 UWP 用户熟悉的长写法是等价的：

```xml
<TextBox Name="other">
<TextBlock Text="{Binding Text, ElementName=other}"/>
```

_Avalonia UI_ 同时支持这两种语法。

## 绑定到祖先元素

你可以使用 `$parent` 语法绑定到目标元素在逻辑控件树中的父元素：

```xml
<Border Tag="Hello World!">
  <TextBlock Text="{Binding $parent.Tag}"/>
</Border>
```

也可以通过在 `$parent` 语法中加索引，绑定到更高层级的祖先元素：

```xml
<Border Tag="Hello World!">
  <Border>
    <TextBlock Text="{Binding $parent[1].Tag}"/>
  </Border>
</Border>
```

索引从零开始，因此 `$parent[0]` 与 `$parent` 等价。

你还可以绑定到最近的某种特定类型祖先元素，例如：

```xml
<Border Tag="Hello World!">
  <Decorator>
    <TextBlock Text="{Binding $parent[Border].Tag}"/>
  </Decorator>
</Border>
```

最后，你也可以把索引和类型组合起来使用：

```xml
<Border Tag="Hello World!">
  <Border>
    <Decorator>
    <TextBlock Text="{Binding $parent[Border;1].Tag}"/>
    </Decorator>
  </Border>
</Border>
```

如果你需要在祖先类型中包含 XAML 命名空间，可以使用冒号把命名空间和类名分开，如下所示：

```xml
<local:MyControl Tag="Hello World!">
  <Decorator>
    <TextBlock Text="{Binding $parent[local:MyControl].Tag}"/>
  </Decorator>
</local:MyControl>
```

如果你要访问父元素 `DataContext` 上的某个属性，就需要使用 `(vm:MyUserControlViewModel)DataContext` 这种类型转换表达式，把它转换为真实类型。否则，`DataContext` 会被视为 `object` 类型，而访问自定义属性时就会产生编译错误。

```xml
<local:MyControl Tag="Hello World!">
  <Decorator>
    <TextBlock Text="{Binding $parent[local:MyControl].((vm:MyUserControlViewModel)DataContext).CustomProperty}"/>
  </Decorator>
</local:MyControl>
```

:::caution
_Avalonia UI_ 也支持 WPF/UWP 的 `RelativeSource` 语法，它的作用看起来类似，但 _并不完全相同_。`RelativeSource` 是基于 _视觉树_ 工作的，而这里介绍的 `$parent` 语法是基于 _逻辑树_ 工作的。
:::

## 另请参阅

- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding paths, modes, and converters.
- [Compiled Bindings](/docs/data-binding/compiled-bindings): Type-safe bindings with compile-time validation.
- [Control Trees](/docs/custom-controls/control-trees): Logical and visual tree structure.
