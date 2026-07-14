---
id: avalonia-xaml
title: Avalonia XAML
description: 学习用于定义 Avalonia 用户界面的 XAML 标记语言。
doc-type: explanation
---

Avalonia 使用 XAML 来定义用户界面。XAML 是一种基于 XML 的标记语言，许多 UI 框架都会使用它。

虽然 XAML 是定义 Avalonia 用户界面最常见的方式，但它并不是必须的。你也可以完全使用 C#、F# 或任意 .NET 语言来开发 Avalonia 应用程序。

:::tip
如果你想了解完全不使用 XAML 构建应用程序的方法，请参阅 [Code-Only UI](/docs/fundamentals/coded-ui)。
:::

:::info
XAML 最初由 Microsoft 为 WPF 开发，后来也被 Silverlight、UWP 等框架采用。Avalonia 使用相同的核心 XAML 概念（声明式标记、对象元素、属性特性、数据绑定和标记扩展），但拥有自己的命名空间和控件库。如果你有 WPF 或 UWP XAML 经验，那么大部分知识都可以直接迁移到 Avalonia。
:::

## AXAML 文件扩展名

其他框架中的 XAML 文件扩展名通常是 `.xaml`，但由于与 Visual Studio 集成时存在一些技术问题，Avalonia 使用了自己的 `.axaml` 扩展名，也就是 “Avalonia XAML”。

## 文件格式

一个典型的 Avalonia XAML 文件如下所示：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="AvaloniaApplication1.MainWindow">
</Window>
```

和所有 XML 文件一样，它有一个根元素。这里的根元素标签 `<Window></Window>` 定义了根对象的类型。在上面的例子中，它对应的是 Avalonia 的一个窗口控件。

上面的示例中有三个值得关注的属性：

* `xmlns="https://github.com/avaloniaui"` - 这是 Avalonia 本身的 XAML 命名空间声明。它是必需的；没有它，文件将不会被识别为 Avalonia XAML 文档。
* `xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"` - 这是 XAML 语言命名空间的声明。
* `x:Class="AvaloniaApplication1.MainWindow"` - 这是上面 `x` 命名空间的一项扩展声明，用来告诉 XAML 编译器去哪里查找与该文件关联的类。这个类定义在 code-behind 文件中，通常使用 C# 编写。

:::info
有关 code-behind 概念的说明，请参阅 [Code-behind](/docs/fundamentals/code-behind)。
:::

## 控件元素

你可以通过添加表示 Avalonia 控件的 XML 元素来组合应用程序界面。元素标签的名称与控件类名相同。

:::info
一个 UI 可以由多种不同类型的控件组成。想了解更多有关 UI 组合的概念，请参阅 [UI composition](/docs/fundamentals/ui-composition)。
:::

例如，下面这段 XAML 会在窗口内容中加入一个按钮：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Button>Hello World!</Button>
</Window>
```

:::info
完整的 Avalonia 内置控件列表请参阅 [Controls reference](/controls)。
:::

## 控件属性

表示控件的 XML 元素可以带有与控件属性对应的特性。你可以通过给元素添加特性来设置控件属性。

例如，要为按钮控件指定蓝色背景，你可以添加 `Background` 属性并将其值设为 `"Blue"`，如下所示：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Button Background="Blue">Hello World!</Button>
</Window>
```

## 控件内容

你可能已经注意到，上面示例中的按钮把内容（字符串 “Hello World”）放在了开始标签和结束标签之间。除此之外，你也可以直接设置内容属性，如下所示：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Button Content="Hello World!"/>
</Window>
```

这是 Avalonia 控件内容模型的一种特性。

## 数据绑定

你通常会使用 Avalonia 的绑定系统，将控件属性连接到底层对象。这个连接通过 `{Binding}` 标记扩展来声明。例如：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Button Content="{Binding Greeting}"/>
</Window>
```

:::info
有关数据绑定原理的更多说明，请参阅 [Introduction to data binding](/docs/data-binding/introduction-to-data-binding)。
:::

## Code-behind 文件

许多 Avalonia XAML 文件还会关联一个 code-behind 文件，通常使用 C# 编写，扩展名为 `.axaml.cs`。

:::info
关于如何使用 code-behind 文件进行开发，请参阅 [Code-behind](/docs/fundamentals/code-behind)。
:::

## XML 命名空间

和其他 XML 格式一样，在 Avalonia XAML 文件中你也可以声明命名空间。XML 命名空间会把元素名和属性名与特定 URI 关联起来，以避免命名冲突，并帮助 XAML 处理器找到文件中各个元素的定义。你可以使用 `xmlns` 属性声明命名空间，并可选择指定前缀。命名空间声明会从父元素继承到子元素，并且从声明位置开始一直在当前包围元素结束前保持有效。

你可以使用 `xmlns` 属性添加命名空间。声明格式如下：

```xml
xmlns:alias="definition"
```

通常的做法是在根元素中一次性定义所有将要使用的命名空间。

在同一个文件中，只有一个命名空间可以不使用别名前缀。其余别名在文件内都必须唯一。

命名空间声明中的定义部分既可以是 URL，也可以是代码定义。两种方式都用于定位文件中元素的具体定义。

:::info
关于命名空间声明如何工作的详细说明，请参阅 [Custom control library](/docs/custom-controls/custom-control-library)。
:::

当 XAML 命名空间属性引用代码时，其定义部分有两种合法语法：

### 使用 using: 前缀

前缀 `using:` 可用于为当前程序集或引用程序集中的命名空间提供别名。两种情况下语法都是一样的。例如：

```xml
xmlns:myAlias1="using:AppNameSpace.MyNamespace"
```
### CLR 命名空间前缀

前缀 `clr-namespace:` 也同样受支持，与 WPF 中的写法一致。不过，具体语法取决于你要设置别名的命名空间位于当前程序集还是引用程序集。

例如，当该命名空间与 XAML 位于同一个程序集时，可以使用如下语法：

```xml
<Window ...
    xmlns:myAlias1="clr-namespace:AppNameSpace.MyNamespace" 
... >
```

如果该命名空间位于另一个被引用的程序集（例如某个类库）中，你必须在描述中额外包含被引用程序集的名称：

```xml
<Window ...
    xmlns:myAlias2="clr-namespace:OtherAssembly.MyNameSpace;assembly=OtherAssembly"
 ... >
```

## 另请参阅

- [Code-Only UI](/docs/fundamentals/coded-ui)
- [Code-behind](/docs/fundamentals/code-behind)
- [UI composition](/docs/fundamentals/ui-composition)
- [Introduction to data binding](/docs/data-binding/introduction-to-data-binding)
