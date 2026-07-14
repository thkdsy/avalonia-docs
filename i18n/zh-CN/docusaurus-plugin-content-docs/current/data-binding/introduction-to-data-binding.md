---
id: introduction-to-data-binding
title: 数据绑定简介
description: 了解 Avalonia 数据绑定如何通过 XAML 标记扩展将 UI 控件连接到数据源。
doc-type: overview
---

import DataBindingDiagram from '@site/src/components/global/DataBindingDiagram/DataBindingDiagram';

Avalonia 使用数据绑定把数据从应用程序对象传递到 UI 控件中，根据用户输入更新应用对象中的数据，并根据用户触发的命令在应用对象上执行操作。

<DataBindingDiagram />

在这种关系中，控件是 **绑定目标**，而对象是 **数据源**。

Avalonia 通过数据绑定系统，根据在 XAML 中声明的简单映射关系完成上述大部分工作，也就是说你无需为此编写大量额外代码。

数据绑定映射通过 XML 来定义，连接的是 Avalonia 控件的属性与应用程序对象的属性。一般语法如下：

```xml
<SomeControl Attribute="{Binding PropertyName}" />
```

这种映射可以是双向的：也就是绑定对象属性的变化会反映到控件上，而控件中的变化（无论由什么原因引起）也会写回到底层对象。例如，一个文本输入框绑定到对象的字符串属性，就是典型的双向绑定。对应的 XML 可能如下所示：

```xml
<TextBox Text="{Binding FirstName}" />
```

如果用户修改了文本框中的内容，那么底层对象的 `FirstName` 属性会被自动更新。反过来，如果底层对象的 `FirstName` 属性发生变化，文本框中显示的文本也会随之更新。

绑定也可以是单向的：也就是绑定对象属性的变化会反映到控件上，但用户不能通过控件反向修改该对象。一个典型例子就是只读的文本显示控件 `TextBlock`。

```xml
<TextBlock Text="{Binding StatusMessage}" />
```

绑定通常与 MVVM 架构模式配合使用，这也是使用 Avalonia UI 进行开发的主要方式之一。

:::info
有关如何在 Avalonia 中使用 MVVM 模式的更多说明，请参阅 [The MVVM Pattern](/docs/fundamentals/the-mvvm-pattern)。
:::

:::info
如果你想了解 _Microsoft_ 中 MVVM 模式的起源和发展背景，可以参阅这篇 [Microsoft Patterns and Practices article](https://msdn.microsoft.com/en-us/library/hh848246.aspx)。
:::

## 绑定模式

绑定可以运行在不同模式下，从而控制数据如何流动：

| 模式 | 说明 |
|---|---|
| `OneWay` | 源变化会更新目标，目标变化不会回传。 |
| `TwoWay` | 源和目标任意一方变化都会更新另一方。 |
| `OneTime` | 只读取一次源值，后续不再跟踪属性变化；但如果 `DataContext` 改变，绑定会重新求值。 |
| `OneWayToSource` | 目标变化会更新源，但源变化不会反向更新目标。 |
| `Default` | 模式由目标属性决定。大多数显示类属性默认是 `OneWay`；像 `TextBox.Text` 这样的可编辑属性默认是 `TwoWay`。 |

```xml
<TextBox Text="{Binding Name, Mode=TwoWay}" />
<TextBlock Text="{Binding Name, Mode=OneWay}" />
```

## FallbackValue 与 TargetNullValue

| 属性 | 说明 |
|---|---|
| `FallbackValue` | 当绑定无法解析时显示的值（例如找不到属性）。 |
| `TargetNullValue` | 当源属性值为 `null` 时显示的值。 |

```xml
<TextBlock Text="{Binding Description, TargetNullValue='No description available'}" />
<Image Source="{Binding AvatarUrl, FallbackValue={StaticResource DefaultAvatar}}" />
```

## 另请参阅

- [Data Context](/docs/data-binding/data-context): Where the data binder gets the data object from.
- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding paths, modes, and converters.
- [The MVVM Pattern](/docs/fundamentals/the-mvvm-pattern): Architectural pattern used with data binding.
