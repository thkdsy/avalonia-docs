---
id: data-context
title: 数据上下文
description: 理解 DataContext 如何为绑定提供默认数据源，以及它如何沿控件树继承。
doc-type: explanation
---

import DataContextOverviewDiagram from '/img/concepts/data-concepts/data-context/data-context-overview.png';
import DataContextTreeSearchDiagram from '/img/concepts/data-concepts/data-context/data-context-tree-search.png';
import DataContextGreetingBindingScreenshot from '/img/concepts/data-concepts/data-context/data-context-greeting.png';
import DataContextPreviewerScreenshot from '/img/concepts/data-concepts/data-context/data-context-previewer.png';

当 Avalonia 执行数据绑定时，它必须先找到一个要绑定到的应用对象。这个绑定目标的位置由 **数据上下文** 表示。

<Image light={DataContextOverviewDiagram} alt="Diagram showing how data context connects controls to view model properties" position="center" maxWidth={400} cornerRadius="true"/>

Avalonia 中的每个控件都有一个 `DataContext` 属性，包括内置控件、用户控件和窗口。

执行绑定时，Avalonia 会从定义绑定的那个控件开始，沿逻辑控件树向上进行层级查找，直到找到一个可用的数据上下文。

<Image light={DataContextTreeSearchDiagram} alt="Diagram showing data context inheritance through the control tree" position="center" maxWidth={400} cornerRadius="true"/>

这意味着：定义在窗口中的控件可以使用窗口的数据上下文；同样地，窗口中的控件里的子控件，也可以继续使用这个窗口的数据上下文。

:::info
有关 Avalonia 控件树以及如何在运行时查看它们，请参阅 [Control trees](/docs/custom-controls/control-trees)。
:::

## 示例

如果你使用 _Avalonia MVVM Application_ 模板创建一个新项目，就可以看到窗口的数据上下文是如何被设置的。打开 **App.axaml.cs** 文件即可看到如下代码：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
    {
        desktop.MainWindow = new MainWindow
        {
            DataContext = new MainWindowViewModel(),
        };
    }

    base.OnFrameworkInitializationCompleted();
}
```

你可以在 **MainWindowViewModel.cs** 文件中找到被设置为窗口数据上下文的对象：

```csharp
public class MainWindowViewModel : ViewModelBase
{
    public string Greeting => "Welcome to Avalonia!";
}
```

在主窗口文件 **MainWindow.axaml** 中，你可以看到窗口内容区域里有一个 `TextBlock`，它的 `Text` 属性绑定到了 `Greeting` 属性。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="using:AvaloniaMVVMApplication2.ViewModels"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="AvaloniaMVVMApplication2.Views.MainWindow"
        Icon="/Assets/avalonia-logo.ico"
        Title="AvaloniaMVVMApplication2">

    <Design.DataContext>
        <vm:MainWindowViewModel/>
    </Design.DataContext>

    <TextBlock Text="{Binding Greeting}" HorizontalAlignment="Center" VerticalAlignment="Center"/>

</Window>
```

当项目运行时，数据绑定器会从文本块开始沿逻辑控件树向上查找，并在主窗口级别找到已经设置好的数据上下文。因此，绑定后的文本会显示为：

<Image light={DataContextGreetingBindingScreenshot} alt="App window showing a greeting bound from the data context" position="center" maxWidth={400} cornerRadius="true"/>

## 设计时数据上下文

你可能已经注意到，在第一次编译这个项目之后，预览窗格中也会显示这个问候语。

<Image light={DataContextPreviewerScreenshot} alt="Design-time preview showing bound data context values" position="center" maxWidth={400} cornerRadius="true"/>

Avalonia 还支持在设计时为控件设置数据上下文。这很有用，因为当你调整布局和样式时，预览窗格就能显示更接近真实的数据。

你可以在 XAML 中看到设计时数据上下文的设置方式：

```xml
<Design.DataContext>
    <vm:MainWindowViewModel/>
</Design.DataContext>
```

:::tip
如果你想进一步了解如何使用设计时数据上下文，请参阅 [XAML preview and design settings](/docs/app-development/xaml-preview-and-design-settings)。
:::

:::info
继续深入学习数据绑定前，通常需要先了解 MVVM 模式的基础概念。相关介绍请参阅 [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)。
:::

## 另请参阅

- [Introduction to data binding](/docs/data-binding/introduction-to-data-binding): Data binding overview.
- [Data binding syntax](/docs/data-binding/data-binding-syntax): Binding paths, modes, and converters.
- [XAML preview and design settings](/docs/app-development/xaml-preview-and-design-settings): Design-time data context configuration.
