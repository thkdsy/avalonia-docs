---
id: namespaces
title: XAML 命名空间
---

XAML 命名空间用于告诉 XAML 引擎去哪里查找标记中引用的类型。每个 Avalonia XAML 文件至少都需要一个命名空间声明才能正常工作。

## 默认命名空间

一个典型的 Avalonia XAML 文件通常会以两个命名空间声明开头：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
</Window>
```

| 声明 | 用途 |
|---|---|
| `xmlns="https://github.com/avaloniaui"` | Avalonia 默认命名空间。它映射到所有核心 Avalonia CLR 命名空间（如 `Avalonia`、`Avalonia.Controls`、`Avalonia.Media`、`Avalonia.Animation` 等）。每个 Avalonia XAML 文件都必须包含它。 |
| `xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"` | XAML 语言命名空间。它提供对 [x: 指令](/docs/xaml/directives) 的访问，例如 `x:Name`、`x:Key` 和 `x:Class`。 |

## 映射到 Avalonia 默认命名空间的 CLR 命名空间

`https://github.com/avaloniaui` 这个 URI 会映射到 Avalonia 程序集中多个 CLR 命名空间。最常用的包括：

- `Avalonia`（基础类型）
- `Avalonia.Controls`（所有标准控件）
- `Avalonia.Controls.Primitives`
- `Avalonia.Controls.Shapes`
- `Avalonia.Controls.Presenters`
- `Avalonia.Controls.Templates`
- `Avalonia.Controls.Documents`
- `Avalonia.Controls.Notifications`
- `Avalonia.Animation`
- `Avalonia.Animation.Easings`
- `Avalonia.Data`（绑定相关类型）
- `Avalonia.Media`（画刷、变换、几何图形）
- `Avalonia.Layout`
- `Avalonia.Styling`
- `Avalonia.Markup.Xaml.MarkupExtensions`
- `Avalonia.Markup.Xaml.Styling`
- `Avalonia.Markup.Xaml.Templates`

## 引用你自己的类型

如果想在 XAML 中使用你自己的类，就需要声明一个映射到 CLR 命名空间的前缀。

### 使用 `using:` 前缀

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="using:MyApp.ViewModels"
        xmlns:controls="using:MyApp.Controls">

    <controls:MyCustomControl DataContext="{x:Static vm:DesignData.MainViewModel}" />
</Window>
```

`using:` 前缀适用于当前程序集或任意已引用程序集中的命名空间。这是推荐的写法。

### 使用 `clr-namespace:` 前缀

同样也支持 WPF 中常见的 `clr-namespace:` 前缀：

**同一程序集：**
```xml
xmlns:local="clr-namespace:MyApp.Controls"
```

**不同程序集：**
```xml
xmlns:ext="clr-namespace:ThirdParty.Controls;assembly=ThirdParty.Controls"
```

:::tip
优先使用 `using:` 而不是 `clr-namespace:`。`using:` 语法更短，而且对于已引用程序集不需要写 `;assembly=` 后缀。
:::

## 定义自定义 XML 命名空间映射

库作者可以使用 `XmlnsDefinition` 程序集特性，把多个 CLR 命名空间映射到同一个 XML 命名空间 URI：

```csharp
// 写在库的 AssemblyInfo.cs 或 Properties 文件中
[assembly: XmlnsDefinition("https://mycompany.com/mylib", "MyLib.Controls")]
[assembly: XmlnsDefinition("https://mycompany.com/mylib", "MyLib.Converters")]
[assembly: XmlnsDefinition("https://mycompany.com/mylib", "MyLib.Panels")]
```

这样一来，库的使用者就可以只写一个命名空间声明：

```xml
<Window xmlns:mylib="https://mycompany.com/mylib">
    <mylib:FancyButton />
</Window>
```

## 常见的命名空间前缀约定

这些前缀在 Avalonia 项目中很常见：

| 前缀 | 常见映射 |
|---|---|
| `x` | XAML 语言命名空间 |
| `d` | `http://schemas.microsoft.com/expression/blend/2008`（设计时） |
| `mc` | `http://schemas.openxmlformats.org/markup-compatibility/2006` |
| `local` | 应用的根命名空间 |
| `vm` | ViewModels 命名空间 |
| `conv` | Converters 命名空间 |

### 设计时命名空间

`d:` 和 `mc:` 命名空间用于启用设计时功能：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d"
        d:DesignWidth="800" d:DesignHeight="450">
</Window>
```

- `d:DesignWidth` 和 `d:DesignHeight` 用于设置 XAML 设计器中的预览尺寸。
- `d:DataContext` 用于为绑定预览设置设计时数据上下文。
- `mc:Ignorable="d"` 告诉运行时忽略所有 `d:` 属性。

## 另请参阅

- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)：XAML 基础与文件结构。
- [x: 指令](/docs/xaml/directives)：XAML 语言指令。
- [自定义控件库](/docs/custom-controls/custom-control-library)：如何通过命名空间映射打包控件。
