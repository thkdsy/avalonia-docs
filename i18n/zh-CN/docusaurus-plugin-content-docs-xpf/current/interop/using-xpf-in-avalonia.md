---
id: using-xpf-in-avalonia
title: 在 Avalonia 中使用 XPF
description: 使用 XpfContainer 在现有 Avalonia 应用程序中承载兼容 WPF 的 XPF 控件。
doc-type: guide
---

本指南将带你在现有的 Avalonia 应用程序中嵌入 XPF（兼容 WPF 的）控件。完成后，你将通过 `XpfContainer` 包装器让一个 XPF `UserControl` 在 Avalonia 窗口中渲染。

## 前提条件

在开始之前，请确保你已经具备：

- 一个现有的 Avalonia 应用程序项目。
- 有效的 XPF 许可证，以及对 XPF SDK NuGet 源的访问权限。有关设置详情，请参阅 [Getting started](/xpf/getting-started)。

## 第 1 步：更新项目文件

将 Avalonia 应用程序中的 SDK 更改为 [使用 XPF SDK](/xpf/getting-started#step-3-use-the-xpf-sdk)：

```xml
<Project Sdk="Xpf.Sdk/1.6.0">
```

接下来，禁用自动 XPF 初始化，这样你就可以控制 XPF 子系统何时启动。将以下属性添加到你的项目文件中：

```xml
<PropertyGroup>
  <DisableAutomaticXpfInit>true</DisableAutomaticXpfInit>
</PropertyGroup>
```

:::tip
必须禁用自动初始化，因为 Avalonia 应用程序负责启动顺序。如果跳过这一步，XPF 可能会在 Avalonia 准备好之前尝试初始化，从而导致运行时错误。更多细节请参阅 [Customizing initialization](/xpf/configuration/customizing-initialization)。
:::

## 第 2 步：添加 XPF 应用程序类

在你的项目中添加一个继承自 `System.Windows.Application` 的 `XpfApp` 类。此类充当 XPF 应用程序对象，是 XPF 资源解析、合并字典以及其他 WPF 基础设施正常工作的必要条件。

```csharp
using System.Windows;

namespace MyAvaloniaApplication;

/// <summary>
/// 表示 XPF 应用程序。
/// </summary>
public partial class XpfApp : Application
{
}
```

:::note
如果你的 XPF 控件依赖应用程序级资源（样式、画刷、转换器），请像在标准 WPF 应用程序中一样，在与此类关联的 `App.xaml` 文件中定义它们。
:::

## 第 3 步：初始化 XPF 应用程序

在你的 Avalonia `App.xaml.cs` 文件中，在 `OnFrameworkInitializationCompleted` 内创建一个 `XpfApp` 实例。你必须在使用任何 XPF 控件之前完成这一步。

```csharp
public override void OnFrameworkInitializationCompleted()
{
    // highlight-start
    new XpfApp();
    // highlight-end

    // Existing Avalonia initialization here
}
```

将 `new XpfApp()` 调用放在方法顶部，位于设置 `MainWindow` 或 `MainView` 之前。如果你在 Avalonia 已经渲染完第一帧之后才初始化 XPF，初始视图树中的任何 `XpfContainer` 实例都可能加载失败。

## 第 4 步：添加一个 XPF UserControl

创建一个包含你要承载的 WPF 兼容内容的 XPF `UserControl`。此控件使用 WPF XAML 命名空间，而不是 Avalonia 命名空间。

```xml
<UserControl x:Class="MyAvaloniaApplication.MyXpfView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             mc:Ignorable="d"
             d:DesignHeight="300" d:DesignWidth="300">
    <Button>Hello XPF!</Button>
</UserControl>
```

```csharp
using System.Windows.Controls;

namespace MyAvaloniaApplication;

public partial class MyXpfView : UserControl
{
    public MyXpfView()
    {
        InitializeComponent();
    }
}
```

:::warning
不要在同一个 XAML 文件中混用 Avalonia 和 WPF 命名空间。XPF `UserControl` 必须使用 WPF 命名空间（`http://schemas.microsoft.com/winfx/2006/xaml/presentation`），而你的 Avalonia 视图必须使用 Avalonia 命名空间（`https://github.com/avaloniaui`）。
:::

## 第 5 步：承载 XPF UserControl

使用 `XpfContainer` 在 Avalonia 控件内部承载你的 XPF 内容。`XpfContainer` 位于 `PresentationFramework` 程序集中的 `Atlantis` 命名空间里。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:MyAvaloniaApplication"
        // highlight-next-line
        xmlns:xpf="clr-namespace:Atlantis;assembly=PresentationFramework"
        x:Class="MyAvaloniaApplication.MainWindow">
    // highlight-start
    <xpf:XpfContainer>
        <local:MyXpfView/>
    </xpf:XpfContainer>
    // highlight-end
</Window>
```

你可以将 `XpfContainer` 放在 Avalonia 可视树中的任何位置：面板、选项卡控件、分割视图或其他任何布局容器内都可以。

## 故障排查

| 症状 | 可能原因 | 修复方法 |
|---|---|---|
| `XpfContainer` 显示为空白 | 控件加载前未实例化 `XpfApp` | 将 `new XpfApp()` 移到 `OnFrameworkInitializationCompleted` 的顶部 |
| 构建错误，引用了缺失的 WPF 类型 | 项目 SDK 未更改为 `Xpf.Sdk` | 检查 `.csproj` 中的 `<Project Sdk="Xpf.Sdk/1.6.0">` 行 |
| XPF 在 Avalonia 准备好之前初始化 | 未设置 `DisableAutomaticXpfInit` | 在项目文件中添加 `<DisableAutomaticXpfInit>true</DisableAutomaticXpfInit>` |
| XPF 控件中的 XAML 命名空间错误 | 混用了 Avalonia 和 WPF 命名空间 | 确保 XPF 控件仅使用 WPF XAML 命名空间 |

## 另请参阅

- [Getting started with XPF](/xpf/getting-started)
- [Customizing initialization](/xpf/configuration/customizing-initialization)
- [Embedding Avalonia in XPF](/xpf/interop/embedding-avalonia-in-xpf)
- [Centralizing multiple XPF projects](/xpf/configuration/centralizing-multiple-xpf-projects)